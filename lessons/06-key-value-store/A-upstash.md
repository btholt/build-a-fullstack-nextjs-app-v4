Now let's look at caching and using a key-value store in your app. I love key-value stores - they're simple tools that yield powerful results. I actually rewrote some of the Redis server as Postgres, just as a fun exercise, [see code here][rtp].

So what is Redis? [I actually teach a fun course on it][redis] if you're interested. It's a key-value store, and that is exactly what it sounds like - you use various keys to access values in a database. An imperfect way to model that in your head is you can think of it as the world's biggest JavaScript object.

```javascript
// just a way to think about it

const redisStore = {};

redisStore.myKey = 5;
console.log(redisStore.myKey); // 5

const redis = require("redis");

await redis.set("myKey", 5);
const val = await redis.get("myKey");
console.log(val); // 5
```

Redis can do a bunch of other stuff like increment, scan, etc. but in essence it's all just a store of values that you access with keys.

> You may see [Valkey][valkey] mentioned in places where Redis is talked about. Long story short: Redis tried to make their license more restrictive and the community revolted and started Valkey (**val**ue **key**). It's a fork of Redis before it messed with their license. Now Redis has eased off but Valkey is still going. Using either is fine.

Generally speaking, it's great for caching and anything you want to access frequently at low latency. Redis's throughput is unbelievably fast, orders of magnitude faster than any SQL or NoSQL database, but you trade off features and frequently some of the guarantees like replication and such.

We're going to use Upstash's managed Redis service, but you could use any number of other ones, I just happen to like Upstash. As a bonus, they have some other cool services like an event notification system and a cron job service.

Head to Upstash, sign up, and click the Redis tab. Create a new database, give it a name, and give it a region preferably close to where your database is. The free plan should be plenty for what we want to do.

Copy the tokens to your .env file. You can use TCP or HTTP. I'm using HTTP with the `@upstash/redis` client, but feel free to use `ioredis` or `node-redis` too.

Install the SDK with `npm i @upstash/redis`

And let's go implement it!

[rtp]: https://github.com/btholt/redis-to-postgres
[redis]: https://btholt.github.io/complete-intro-to-databases/key-value-store
[valkey]: https://valkey.io/
