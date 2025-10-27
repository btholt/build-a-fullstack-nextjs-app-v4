Now we've used server helpers to fetch data, let's use form actions to submit data.

> If server actions are new to you, [check out Intermediate React v6][v6] - we cover these with Next.js in depth.

We're going to be editing app/actions/articles.ts. Let's start with creating a new article.

```typescript
// at top
import { eq } from "drizzle-orm";
import db from "@/db/index";
import { articles } from "@/db/schema";

// replace create article
export async function createArticle(data: CreateArticleInput) {
  const user = await stackServerApp.getUser();
  if (!user) {
    throw new Error("❌ Unauthorized");
  }
  console.log("✨ createArticle called:", data);

  const response = await db.insert(articles).values({
    title: data.title,
    content: data.content,
    slug: "" + Date.now(),
    published: true,
    authorId: user.id,
  });

  return { success: true, message: "Article create logged" };
}
```

- Looks fairly similar to our reads, only here we just use `insert` instead of `select`.
- We're only checking that the user is logged in, not that the correct user can edit the article. We'll implement proper authorization in the next section.

Let's do update and delete.

```typescript
export async function updateArticle(id: string, data: UpdateArticleInput) {
  const user = await stackServerApp.getUser();
  if (!user) {
    throw new Error("❌ Unauthorized");
  }

  // TODO: Replace with actual database update
  console.log("📝 updateArticle called:", { id, ...data });

  const response = await db
    .update(articles)
    .set({
      title: data.title,
      content: data.content,
    })
    .where(eq(articles.id, +id));

  return { success: true, message: `Article ${id} update logged` };
}

export async function deleteArticle(id: string) {
  const user = await stackServerApp.getUser();
  if (!user) {
    throw new Error("❌ Unauthorized");
  }

  console.log("🗑️ deleteArticle called:", id);

  const response = await db.delete(articles).where(eq(articles.id, +id));

  return { success: true, message: `Article ${id} delete logged (stub)` };
}
```

Nothing groundbreaking here! Same motion, just with `update` and `delete`!

[v6]: https://holt.fyi/intermediate-react
