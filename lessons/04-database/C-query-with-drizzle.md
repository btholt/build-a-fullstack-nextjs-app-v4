Let's start by doing the SELECTS first. Our app is already pulling dummy data via a server helper function, so this makes it pretty simple as we only need to update this server helper as opposed to doing it in the code (and makes it super testable!)

Let's open src/lib/data/articles.ts.

```typescript
// replace everything but the getArticlesById function - we'll do that in a sec
import db from "@/db/index";
import { articles } from "@/db/schema";
import { eq } from "drizzle-orm";
import { usersSync } from "@/db/schema"; // <- this is different from the video - it's the new path, or it can be combined with the above import

export async function getArticles() {
  const response = await db
    .select({
      title: articles.title,
      id: articles.id,
      createdAt: articles.createdAt,
      content: articles.content,
      author: usersSync.name,
    })
    .from(articles)
    .leftJoin(usersSync, eq(articles.authorId, usersSync.id));
  return response;
}
```

- We're first importing our db client. This is what will actually connect to the database.
- We then import the articles schema. Why? This is how you reference which table you want to query. We want the articles table, so we import that schema.
- In the function, we run a select, choose what columns we want, tell it which table it's coming from, and then join in the author's name (because we want to say the article was written by Bob, not "user-12334")
- We added the where clause and used `eq` which, as you may imagine, checks for equality. Here we're saying we want to do the join on where the authorId in the article table is the same as the usersSync id so we can get a name instead of an ID.
- And then we return the array of rows! That's it!
- This does not paginate. But it's very easy to do with either [cursors][cursors] or just using limit() and offset like normal SQL. Feel free to implement this yourself!

[cursors]: https://orm.drizzle.team/docs/guides/cursor-based-pagination

This should make articles on the home page pull from the database! Hooray! Let's make the articles page pull from it too.

```typescript
export async function getArticleById(id: number) {
  const response = await db
    .select({
      title: articles.title,
      id: articles.id,
      createdAt: articles.createdAt,
      content: articles.content,
      author: usersSync.name,
      imageUrl: articles.imageUrl,
    })
    .from(articles)
    .where(eq(articles.id, id))
    .leftJoin(usersSync, eq(articles.authorId, usersSync.id));
  return response[0] ? response[0] : null;
}
```

- Here we just added the where filter to filter it down to one record, the correct article ID.
- Drizzle doesn't have a select one function, but it's indexed so it's not a big deal.
- You'll notice that the two functions are quite similar, and there might be some DRY part of your brain twitching here, but I'd say calm it and tell it that it's fine that we repeated ourselves. These two functions accomplish different things, and it's possible we could want to optimize them individually the future. There's no need for complex abstractions here, just have some WET (write everything twice/thrice) code here, no big deal.
- By the end of this project this will be the only data helper, but I like this pattern of having helpers to call database functions instead of just having the raw DB queries in your React UI - makes it a bit more centrally maintained that the UI access patterns and the underlying DB are maintained separately and can be modified either way without disturbing the other too much.

Awesome! That's our SELECTs! Let's go do our writes!
