Now that we have both Neon Auth and the database, we can implement authorization (often abbreviated as authZ vs authN which is short for authentication).

Because it confuses a lot of people, let's quickly disambiguate the two. Authentication is logging in, logging out, and signing up. It's you handshaking to the service "this is who I am" via social login, username-and-password, etc. Authentication answers the question **who is it**? Authorization is taking your identity and asking the question **what are you allowed to do?** I can be logged in to Facebook but I can't see everyone's DMs. Why? Because I am not authorized to do so. However you are authorized to see your own DMs. And possibly the admins / moderators of Facebook can see them too, because they may be authorized to see _anyone_'s DMs.

We already did authentication when we implemented Stack Auth. But we haven't done anything with authorization. We just said "if you're logged in you're authorized to do anything". Let's go make it so you can only edit your own posts.

> You'll see authZ and authN everywhere. Generally speaking when people write "Auth" they mean either just authN or both authZ and authN.

Go to the db folder and create a file called authz.ts and put this in there

```typescript
import { eq } from "drizzle-orm";
import db from "@/db/index";
import { articles } from "@/db/schema";

export const authorizeUserToEditArticle = async function authorizeArticle(
  loggedInUserId: string,
  articleId: number
): Promise<boolean> {
  const response = await db
    .select({
      authorId: articles.authorId,
    })
    .from(articles)
    .where(eq(articles.id, articleId));

  if (!response.length) {
    return false;
  }

  return response[0].authorId === loggedInUserId;
};
```

This takes in an article ID and a user ID and returns if they are able to edit that article or not. Then we can reuse this helper in several places. Later, if we ant to add an editor role that can edit anything, we could just add it here and have it work everywhere. That's the idea.

There's two ways I could have written this. I chose to write the authZ part in TypeScript, `response[0].authorId === loggedInUserId`. We could have written this as SQL, `and(eq(articles.id, articleId), eq(articles.authorId, loggedInUserId))` and let the database done the checking instead of us in TypeScript. Inevitably someone will take exception to the fact this was written this way, so let me explain myself.

- Letting the database do it will be ever-so-slightly more performant, probably, which is why some people were prefer that way. You may need to add an index to accomplish that, but that's not really a big enough deal for that to be a reason to not do it.
- I like doing it in code because I find the code more readable, and the performance hit is so minimal that I choose to value what I find more readable over what could save a millisecond.
- Doing it in code would make it easier to refactor later to add other authZ logic here.

Okay, we have authZ logic, now go back to your articles.ts in your actions directory and let's implement it there

```typescript
// at top
import { authorizeUserToEditArticle } from "@/db/authz";

// put after authorized check in delete and update function
if (!(await authorizeUserToEditArticle(user.id, +id))) {
  throw new Error("❌ Forbidden");
}
```

That's it!

What about error handling? There's a couple ways of handling errors with server actions and I find this to be the most straightforward: just throw an error and catch it on the client. This isn't like a normal API with status codes and such - with a server action there's no public reusable API, just really remote code execution. You could also return status codes in the replies if you want to mimic that aspect of API calls, but that point you may almost just be better off making real APIs. The point of server actions to have them feel more like code than remote API invokations.
