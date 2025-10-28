> Just to be clear again, I work at Neon (now owned by Databricks). Feel free to use any other Postgres provider or even to run it locally with Docker.

Let's get going with Neon! Neon is a serverless Postgres provider that has some awesome features around it. It's serverless because you don't have to scale it, Neon will. You can set a min and max CPU and even tell it to scale to zero when it's idle to save you money. It has a pretty generous free tier that will more than cover what you'll need for this course. Since we've already signed up for it and copied the environmental variables into our project, we can get going right away. If you need to, head back to the Auth section to see how to set up Neon.

> I'll also mention this is definitely not a Postgres class so we won't spend too much time on the SQL itself. If you want that, [try my SQL class][sql] to get started with SQL.

The basic idea here though is that Postgres is a repository for your data, made to scale to as big as your data needs. You can think of a database like a massive spreadsheet - it has tables, rows, and columns. Those rows, columns, and rules can have data types, functions, validation, and all sorts of other logic on top of them. Postgres is one of the many relational databases that speak SQL and it happens to be the fastest growing by far. After having worked on or with MySQL, SQLite, MongoDB, and others, I'm convinced that unless you have some niche use case you should always use Postgres.

For our app, we want to add the database into a few places: reading to get the articles out of the database, and writing edits and updates to the database. That'll be enough for our use case.

> One cool feature of using Neon Auth is that your user data gets copied into your database, making it very easy to query instead of having to call an API to get it.

We could write raw queries directly (and this case that is more than fine) but I want to show you how I would do it if we were making this into a long-term project or company. I'd use Drizzle.

[sql]: https://holt.fyi/sql
