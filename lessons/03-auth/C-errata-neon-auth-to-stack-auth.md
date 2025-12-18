---
title: "Errata: Neon Auth to Stack Auth"
---
## What Changed

Neon Auth (powered by Stack Auth) has been deprecated. The `usersSync` table from drizzle-orm/neon that auto-synced users no longer works.

## Solution 

Sign up for [Stack Auth](https://stack-auth.com/) directly and sync users manually when they create articles.

Follow the steps [in the project repo][errata] to migrate from Neon Auth to Stack Auth.

[errata]: https://github.com/btholt/fullstack-next-wiki/blob/main/ERRATA.md
