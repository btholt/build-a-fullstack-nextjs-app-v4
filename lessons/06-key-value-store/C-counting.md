What if we wanted to track pageviews on each article to show social proof that people are using the wikis? We could do that in Postgres but that's a lot of writes for not very important data. That's actually what Redis excels at, so let's go implement that!

Make a new file at src/apps/actions/pageviews.ts and put this in there

```typescript
"use server";

import redis from "@/cache";

const keyFor = (id: number | string) => `pageviews:article:${id}`;

export async function incrementPageview(articleId: number) {
  const articleKey = keyFor(articleId);
  const newVal = await redis.incr(articleKey);
  return +newVal;
}
```

- incr both increments the existing number _and_ it returns the current value.

Now let's add it to our wiki-article-viewer.ts

```typescript
// top
import { incrementPageview } from "@/app/actions/pageviews";

// near top of React component
useEffect(() => {
  async function fetchPageview() {
    const newCount = await incrementPageview(article.id);
    setLocalPageviews(newCount ?? null);
  }
  fetchPageview();
}, []);

// under the Article badge
<div className="ml-3 flex items-center text-sm text-muted-foreground">
  <Eye className="h-4 w-4 mr-1" />
  <span>{localPageviews ? localPageviews : "—"}</span>
  <span className="ml-1">views</span>
</div>;
```

That's it! Now we should have a very cool page counter, all being powered by Redis.

> 🏁 This is the [06-caching][checkpoint] checkpoint. Open that folder in the sample project repo to go to where we are as of right here.

[checkpoint]: https://github.com/btholt/fullstack-next-wiki/tree/main/06-caching
