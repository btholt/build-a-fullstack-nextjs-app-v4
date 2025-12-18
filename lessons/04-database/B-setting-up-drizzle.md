We are going to get started writing to our database using Drizzle. Drizzle is a phenomenal ORM, which stands for object-relational mapping. In reality it just means a it's a software package where you define the shape of your data, and it takes care of managing and querying the database for you. Instead of writing raw SQL statements, you write code that then gets translated into SQL for you.

In the past I did not recommend using an ORM (I think you can find this in my previous Frontend Masters courses!) Why not? I had some pretty bad experiences over the year using ORMs in the earlier days of my career (mostly in PHP and Java.) I'd start using an ORM and it would be amazing: it made it easy to get started, to do basic selects and inserts, and the general 95% use case (and I'll say generally speaking, writing this sort of SQL is not hard.) The problem came when you needed to do more advanced querying that the designers of the ORM didn't anticipate. All the sudden what was helping you work faster was a huge impedance on you doing what you want to do. You're suddenly fighting the framework instead of being helped by it. This happened frequently enough that I decided I'd rather just write SQL, and I did that for most of my career (I also like SQL, but it took a lot of practice for me to say that.)

So why now? Why do I like Drizzle instead of choosing to just to continue to do raw SQL?

- Its design is very SQL-ish. A lot of other ORMs try to hide SQL from you and in the process make it hard when you need to do SQL-ish things. that's probably my biggest complaint about other ORMs and I _don't_ have that about Drizzle.
- TypeScript support, and that's the biggest reason _to_ use Drizzle. When you describe something in Drizzle, all the sudden you have amazing TypeScript support for all your database queries. Otherwise you'd be stuck writing all these types yourself and with Drizzle you just don't have to.
- They even go one step further and they make little packages for each database provider. For Neon, we have all the Neon Auth tables built into the Drizzle package so you don't need to write those types; they're just built into Drizzle. So cool!
- The OSS team is also super nice and helpful.

So let's get started! We're going to need a few packages

```bash
npm i drizzle-orm @neondatabase/serverless dotenv
npm i -D drizzle-kit drizzle-seed
```

- The ORM package is package that you'll actually use in your codebase.
- The drizzle-kit package is all the CLI commands you need to run Drizzle. So creating migrations, running migrations, etc.
- We could use the normal pg and postgres.js packages, and in many cases you might want to. These use TCP for their connections and support connection pooling that leave connections open which means lower-latency and generally faster connections. However initial connections for these sorts of packages take a while and really aren't a good fit for things like serverless environments where connections will be spinning up and spinning down frequently.
- We're going to use the Neon serverless driver. This allows us to do SQL over either HTTP or WebSockets (and we're going to do HTTP.) Honestly if we were going to scale up this project, we'd probably want to do the TCP drivers as it might make more sense, but I usually get started with the serverless driver and switch when I see it being helpful. Both work really well.
- Doing Neon over HTTP is perfectly suited for Vercel's serverless architecture, but it does carry some performance overhead. If you're really performance sensitive or doing transactions is really important to you, we'd need to re-architect this to happen over websockets. But we don't so this works!
- We're also install drizzle-seed which makes seeding your Drizzle database very easy.

Okay, let's start making our database work. Normally you'd need to go to Neon.com and create your project and get your DATABASE_URL and put that in your .env file, but we did that as part of setting up auth. So let's go ahead and start with our config.

Go create in the root of the project drizzle.config.ts. Put in there

```typescript
import "dotenv/config";
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  out: "./drizzle",
  schema: "./src/db/schema.ts",
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

This is just some basic config for Drizzle, nothing of note.

Now go create src/db as a folder. Put in there schema.ts

> 🚨 The recorded version of this course didn't have you set up the users table. Since this part of Neon Auth doesn't exist and Stack Auth doesn't have it, we are going to build a simple version of it ourselves. We are just going to handle the account creation part. If you want to later go and add more thorough syncing, it'd be a good exercise.

```typescript
import { pgTable, serial, text, timestamp, boolean } from "drizzle-orm/pg-core";
// import { usersSync } from "drizzle-orm/neon"; <- this doesn't work anymore

export const articles = pgTable("articles", {
  id: serial("id").primaryKey(),
  title: text("title").notNull(),
  slug: text("slug").notNull().unique(),
  content: text("content").notNull(),
  imageUrl: text("image_url"),
  published: boolean("published").default(false).notNull(),
  authorId: text("author_id")
    .notNull()
    .references(() => usersSync.id),
  createdAt: timestamp("created_at", { mode: "string" }).defaultNow().notNull(),
  updatedAt: timestamp("updated_at", { mode: "string" }).defaultNow().notNull(),
});

const schema = { articles };
export default schema;

export type Article = typeof articles.$inferSelect;
export type NewArticle = typeof articles.$inferInsert;

// add this
export const usersSync = pgTable("usersSync", {
  id: text("id").primaryKey(),  // Stack Auth user ID
  name: text("name"),
  email: text("email"),
});
export type User = typeof usersSync.$inferSelect;
```

- While this is a lot of new code for you, if you know SQL it should all look _super_ familiar to you. We're basically doing `CREATE TABLE` commands in code. We're describing what data types we want and what constraints we want (like notNull or unique).
- usersSync from the neon portion of the drizzle-orm package describes the users table from Neon Auth. It's a table that already exists, and we already have all the types and such from Drizzle, made by the Neon and Drizzle team. Pretty cool that it already exists!
- `references` sets up a foreign key. That means the authorId references the id key in the usersSync table.
- What's nice is we're not stuck calling "created_at" using snake case in JavaScript. Drizzle makes it easy for us to define our own alias of what we want to call it in code versus what it's called in the database. This was particularly helpful in a codebase I was working in where the actual names of the columns were very long and annoying due to being apart of another system but we could call it whatever we wanted in JS code.
- `$inferSelect` and `$inferInsert` are probably two of the coolest black magic features in code I've ever used. It takes the database shape that we set up for the articles tables and turns it into a TypeScript type. We write the code once and we get both the TypeScript types and the database ORM to use. Amazing. If you're writing raw SQL, you need to author and maintain those types yourself.

> 🚨 We need to add this helper (which is not in the recorded version) to add sync'ing of our users to our users table

```typescript
import db from "@/db/index";
import { usersSync } from "@/db/schema";

type StackUser = {
  id: string;
  displayName: string | null;
  primaryEmail: string | null;
};

/**
 * Ensures the Stack Auth user exists in our local users table.
 * Call this before creating articles to ensure the foreign key reference works.
 */
export async function ensureUserExists(stackUser: StackUser): Promise<void> {
  await db
    .insert(usersSync)
    .values({
      id: stackUser.id,
      name: stackUser.displayName,
      email: stackUser.primaryEmail,
    })
    .onConflictDoUpdate({
      target: usersSync.id,
      set: {
        name: stackUser.displayName,
        email: stackUser.primaryEmail,
      },
    });
}
```

That's it! Now we have a schema that we can use to create our database connection. Go create an index.ts file in the same directory.

```typescript
import { neon } from "@neondatabase/serverless";
import { drizzle } from "drizzle-orm/neon-http";
import * as schema from "@/db/schema";
import "dotenv/config";

const sql = neon(process.env.DATABASE_URL!);
const db = drizzle(sql, { schema });

export default db;
```

This is just setting up the ORM to be ready to be used by your code. If you were using the pg or postgres packages, you'd set those up here instead. It's really easy to swap in the future - no other code needs to change.

Okay, now let's create our first migration with generate.

```bash
npx drizzle-kit generate
```

This actually creates the drizzle directory in the root of your project (check this code in) and spits out some meta data and raw SQL files. Feel free to read these (I like Drizzle because these are usually pretty readable) but never, ever, ever modify these by hands. You are setting yourself up for major issues if you do because Drizzle assumes it does all of these and will not respect any handcrafted code you chuck in here.

> Note that this creates the migrations but does not apply them. Your Neon database will still be empty until you run migrate.

Now let's run it.

```bash
npx drizzle-kit migrate
```

This applies what we made with generate. And now you can see the empty tables in Neon. Pretty cool, right!?

> You can also run `npx drizzle-kit push` to just yolo apply whatever schema you have at the moment. This is nice when you're making changes and you just want to apply the new schema and aren't ready to codify what you have as code.

I wrote a little seed script for you to have some database. **Note: you must have at least one user signed up via Neon Auth or this does not work**. Either sign up via your website or add one manually via the Neon console.

[Copy and paste all of this code][seed] into db/seed.ts.

Let's make all of these npm scripts.

```json
// the end of your scripts in your package.json
"db:seed": "tsx src/db/seed.ts",
"db:generate": "drizzle-kit generate",
"db:migrate": "drizzle-kit migrate"
```

We'll need tsx for this as well (run TypeScript files as Node, just makes it easy) so run `npm i -D tsx`

So feel free to run `npm run db:seed` now and you should have five articles in your database now to play with.

Awesome. Let's go _use_ these now.

[seed]: https://github.com/btholt/fullstack-next-wiki/tree/main/04-database/src/db/seed.ts
