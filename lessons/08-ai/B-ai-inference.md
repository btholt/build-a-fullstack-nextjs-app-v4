---
title: AI Inference
---

Let's build!

In your src directory, create an ai directory. In there put a summarize.ts file and put this in there

```typescript
import { generateText } from "ai";

export async function summarizeArticle(
  title: string,
  article: string
): Promise<string> {
  if (!article || !article.trim()) {
    throw new Error("Article content is required to generate a summary.");
  }

  const prompt = `Summarize the following wiki article in 1-2 concise sentences. Focus on the main idea and the most important details a reader should remember. Do not add opinions or unrelated information. The point is that readers can see the summary a glance and decide if they want to read more.\n\nTitle:\n${title}\n\nArticle:\n${article}`;

  const { text } = await generateText({
    model: "openai/gpt-5-nano",
    system: "You are an assistant that writes concise factual summaries.",
    prompt,
  });

  return (text ?? "").trim();
}

export default summarizeArticle;
```

I love the AI SDK. It's a much nicer experience than say the OpenAI SDK's rituals can be, in my opinion.

What's cool about the AI SDK is you can easily switch it to use Anthropic, OpenAI, etc. directly instead of going through Vercel. Smart marketing on their part because it means you can use their SDK no matter what and it's easy to switch to their AI Gateway

Let's say you wanted to use Anthropic directly. Here's what you'd do.

> I'm going to use the Vercel AI Gateway so only do these next steps if you want to switch. I just wanted to show you how easy it is to switch.

```bash
npm i @ai-sdk/anthropic
```

```typescript
// at top
import { anthropic } from "@ai-sdk/anthropic";

// replace model
model: anthropic("claude-haiku-4-5"),
```

That's it! Now instead of going to Vercel's AI Gateway, you're going directly Anthropic. This would be helpful if you already had money on your Anthropic account and didn't want to fill up a Vercel account. We're not doing this, and we're just going to use Vercel's AI Gateway.

> I chose `openai/gpt-5-nano` because it was cheapest of the frontier models as of writing. Summarizing is the most basic AI task and basically any model can do it pretty well. Feel free to experiment here. [Here are the models supported][gateway]. Also, just throwing out that the prompt could be improved. This will do the job but feel free to iterate.

Okay, let's add it to our server actions. Update actions/articles.ts

```typescript
// at top
import summarizeArticle from "@/ai/summarize";

// replace db call in create
const summary = await summarizeArticle(data.title || "", data.content || "");
const response = await db.insert(articles).values({
  title: data.title,
  content: data.content,
  slug: "" + Date.now(),
  published: true,
  authorId: user.id,
  imageUrl: data.imageUrl ?? undefined,
  summary, // add
});

// replace db call in update
const summary = await summarizeArticle(data.title || "", data.content || "");
const response = await db
  .update(articles)
  .set({
    title: data.title,
    content: data.content,
    imageUrl: data.imageUrl ?? undefined,
    summary: summary ?? undefined, // add
  })
  .where(eq(articles.id, +id));
```

- We're just using our new module and inserting into the database.
- The summary _and_ the title should always be there from the frontend. If it wasn't then we'd need to select the necessary fields from the database and then do the summary. You could be more defensive here, but I'm keeping it simple.

Let's go add it to the database. Go to db/schema.ts

```typescript
// add the row
summary: text("summary"),
```

Now run these CLI commands

```typescript
npm run db:generate
npm run db:migrate
```

> You may have a problem with untracked migrations, particularly if you played around with `drizzle-kit push`. There's a bunch of ways to handle this, but I found the easiest way to just drop the articles table and `npm run db:migrate` again and then run `npm run db:seed` again.

Awesome, now, finally, let's add it to the select in data/articles.ts

```typescript
// replace `getArticles` field
summary: articles.summary,
```

Now we can see the summary on the home page for new and edited articles!

[gateway]: https://vercel.com/ai-gateway/models
