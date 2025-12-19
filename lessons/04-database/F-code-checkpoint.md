---
title: "Errata: Code Checkpoint"
---
## `04-database` Code Checkpoint

If you get stuck after the **Authentication** or **Neon Postgres Database** sections, grab the [04-database solution code](https://github.com/btholt/fullstack-next-wiki/tree/main/04-database) and confirm you've done these steps:

### Created a Neon account
1. Head to [Neon.com](https://neon.com)
1. Sign up for an account
1. Create a new project - use the most recent version of Postgres and it doesn't matter what cloud or region you choose
1. Copy the connection strings and paste it into a .env file at the root of your project.

### Created a Stack Auth account
1. Sign up for [Stack Auth](https://stack-auth.com/)
1. Create a new project
1. Add these credentials to .env.local:
  - `NEXT_PUBLIC_STACK_PROJECT_ID`
  - `NEXT_PUBLIC_STACK_PUBLISHABLE_CLIENT_KEY`
  - `STACK_SECRET_SERVER_KEY`

### Seeded the DB and Run the `04-database` Code

```bash
cd 04-database # using the 04-database code
npm i # install the dependencies
npm run db:generate # generate a DB migration (you might be up to date)
npm run db:migrate # push the migration 
npm rn db:seed # seed the DB with a Seed User and Articles
npm run dev #open localhost:3000 and you should have a working application
```
