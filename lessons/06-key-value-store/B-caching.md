We are going to pretend for a second that our initial page load is reeeeeally slow or expensive (it's neither at the moment) because it's querying the database with a heavy query. If that was true, we'd want to cache our database response for that. Let's go do that.

> Premature optimization kills startups. Generally speaking, don't do cache something until it's proven to be a problem. Whenever you try to guess what the scaling problems are going to be, you're usually wrong, and now you have two problems: an unnecessary hack and an actual scaling problem.

Let's first make a client that we can use anywhere. Make a `cache` directory in app and make an index.ts file and put this in there.

```typescript
import { Redis } from "@upstash/redis";
const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL,
  token: process.env.UPSTASH_REDIS_REST_TOKEN,
});

export default redis;
```

From there in lib/data/articles.ts, add this

```typescript
// at top
import redis from "@/cache";

// top of getArticles
const cached = await redis.get("articles:all");
if (cached) {
  console.log("🎯 Get Articles Cache Hit!");
  return cached;
}

// above return statement
console.log("🙅‍♂️ Get Articles Cache Miss!");
redis.set("articles:all", response, {
  ex: 60, // one minute
});
```

Now we cache the results of the getArticles call for one minute, meaning that we should really only see a little load on the database even under heavy traffic - most of those reads can just go straight to Redis and skip your database all together! Pretty cool!

> Could this be better? Definitely. Instead of just relying on the cache to expire and then reset it in code, we could set the cache to expire in 20 minutes, and then have the cache be refreshed every minute via a job. That way we guarantee that the database is going to be being called every minute with fresh data and no "thundering herd" problems. Thundering herd is what they call it when your cache expires and a ton of traffic spikes on your database all at once causing instability. But this will work for now and it's nice-and-simple.

What if someone creates a new article? We want that to show instantly. So let's go clear the cache on article creation.

```typescript
// at top
import redis from "@/cache";

// bottom of createArticle, above return statement
redis.del("articles:all");
```

That's it! We clear the cache on new article creation so it's always immediately available, but otherwise we assert that the data being a minute old is fresh enough.
