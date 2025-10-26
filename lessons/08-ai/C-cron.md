So, now we have a problem - we have some records that have AI summary records, and some that don't. Maybe we make those summary updates low priority, or maybe they fail on some normal interval. What can we do about that? We could make it check on reads but that slows down a critical path. What we could do better is schedule a job to run on some reoccuring basis, essentially a [cron job][cron].

Vercel has a very easy way to do cron jobs for Next.js apps. You just define an API function and Vercel will call the API function for you. Vercel will put in a special variable so you can make sure it's only Vercel that will call the function too (we don't want random people invoking our jobs.)

Let's do it! Let's make a job that runs weekly to make sure that all items in the database have summaries.

> Why weekly? Every time this runs it'll wake your Vercel and Neon instances, costing you money or free tier credit. I chose weekly because adding a few minutes week isn't too bad. If you ran this every minute you'd eat through both Vercel and Neon's free tiers.

In src/app, create a new folder, `api`. In the api directory, make a new directory, `summary`. In there, create a new file, `route.ts`.

In there put:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { eq, isNull } from "drizzle-orm";
import summarizeArticle from "@/ai/summarize";
import db from "@/db";
import { articles } from "@/db/schema";
import redis from "@/cache";

export async function GET(req: NextRequest) {
  if (
    process.env.NODE_ENV !== "development" &&
    req.headers.get("authorization") !== `Bearer ${process.env.CRON_SECRET}`
  ) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  // Find articles that don't yet have a summary
  const rows = await db
    .select({
      id: articles.id,
      title: articles.title,
      content: articles.content,
    })
    .from(articles)
    .where(isNull(articles.summary));

  if (!rows || rows.length === 0) {
    return NextResponse.json({ ok: true, updated: 0 });
  }

  let updated = 0;
  console.log("🤖 Starting AI summary job");

  for (const row of rows) {
    try {
      const summary = await summarizeArticle(row.title ?? "", row.content);

      if (summary && summary.trim().length > 0) {
        await db
          .update(articles)
          .set({ summary })
          .where(eq(articles.id, row.id));
        updated++;
      }
    } catch (err) {
      // log and continue with next article
      console.error("Failed to summarize article id=", row.id, err);
      continue;
    }
  }

  if (updated > 0) {
    // Clear articles cache used by getArticles
    try {
      await redis.del("articles:all");
    } catch (e) {
      console.warn("⚠️ Failed to clear articles cache", e);
    }
  }

  console.log(`🤖 Concluding AI summary job, updated ${updated} rows`);

  return NextResponse.json({ ok: true, updated });
}
```

- It now works in dev so we can just hit `localhost:3000/api/summary` in a browser and it'll run
- In prod it'll check the CRON header and if it doesn't match it won't run.
- Beyond that, we're just reading from the DB and updating rows that don't have summaries
- We also clear cache if any of the articles get updated

Make a new file called `vercel.json` and we'll have it run once a week.

```json
{
  "crons": [
    {
      "path": "/api/summary",
      "schedule": "0 0 * * 0"
    }
  ]
}
```

This will run every Sunday at midnight UTC. Feel free to make it whenver you want. If you need help [cron.guru][guru] is very helpful.

And there you go! Now you can have AI summaries running once a week, and in general you now how to do jobs with Vercel and Next.js. This is nice but it applies only to Vercel. Generally speaking I've usually done these sorts of jobs with serverless functions like Azure Functions or AWS Lambdas as those are easy to schedule.

You also learned how to make API endpoints with Next.js. These days I think unless you're making endpoints available to outside users or mobile apps, it's a bit of an antipattern to make API endpoints as you should just be using React Server Components do all the connecting between clients and servers.

Lastly we learned how to do migrations with Drizzle. While this is a pretty simple example of it, this is how you do it - just modify your schema files and let Drizzle handle the rest!

> 🏁 This is the [08-ai][checkpoint] checkpoint. Open that folder in the sample project repo to go to where we are as of right here.

[checkpoint]: https://github.com/btholt/fullstack-next-wiki/tree/main/08-ai
[cron]: https://btholt.github.io/complete-intro-to-linux-and-the-cli/cron
[guru]: https://crontab.guru/#0_0_*_*_0
