Let's get started.

Go ahead and create a new Next.js app. I normally do this through [create-next-app][cna]. This is a great template for getting started and is ideally situated for our tech stack, namely Vercel.

Run the following:

```bash
npx create-next-app@15.5.3 wikimasters
```

I choose all the defaults except I chose Biome instead of ESLint. Biome is a very cool project that is both a linter and a formatter.

This will create a new Next.js app for us and ask us a few questions. Normally you would do `@latest` instead of the version I chose, but for our purposes I want it match as close to my environment as possible so the code continues to work long-term.

Let's make sure all our new code works

```bash
npm run lint
npm run format
npm run dev
```

This will lint, format, and then eventually run the server for you. As this isn't an intro to Next course, not much of this should be new.

> Good idea to install the [Biome.js VS Code extension][biome]. Very helpful so you don't need to wait unti CI/CD to find out you have issues

There's a good chance there will be some drift in how Create Next App works between when I write this and when you watch it, so feel free to just clone the repo here and start from 00-start to make sure we're totally on the same page.

> 🏁 This is the [00-start][checkpoint] checkpoint. Open that folder in the sample project repo to go to where we are as of right here.

[checkpoint]: https://github.com/btholt/fullstack-next-wiki/tree/main/00-start
[cna]: https://nextjs.org/docs/app/api-reference/cli/create-next-app
[biome]: https://marketplace.visualstudio.com/items?itemName=biomejs.biome
