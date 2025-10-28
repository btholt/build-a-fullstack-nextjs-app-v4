Let's talk about the CD in CI/CD. How do we get our code from code to deployed? GitHub and Vercel!

There are a trillion ways to do this but in this I'd say let's just lean into Vercel - they're the Next.js people, might as well let them do their thing. We _could_ build our app on GitHub and then take the loose files and deploy those to Vercel, but in this case Vercel is so optimized for Next.js I'm just inclined to let them work their magic.

The cool is it's kinda already set up. Vercel does a really good job making this easy. It can be a huge pain in the ass to get it going on other platforms, and Vercel just makes it work for the average case so well. We really even get preview deployments sorta for free.

What will happen now, by default, is that Vercel will deploy everything in master automatically. There's a lot of other ways to do this, but I generally like the idea "every commit that makes it into master should be able to make it to production." With this idea, this deployment pipeline works great.

By default, GitHub will also not let you deploy unless your tests pass (assuming you've set up Actions, which we will here shortly.)

Let's do something fun. Open a PR for something visual on the site : change a button color, add big text, anything. But do it in a branch, and then open a PR on GitHub for it.

I just made one and it'll look something like this: [make everything red PR][red]. You won't be able to see the PR as you're not on my Vercel team, but you can see Vercel deploys it for me, makes a preview available, that once I've merged we can then promote to production. This will also run the CI (which we'll set up in the next section.)

Another cool thing: you can go in to the preview deploy and leave blocking comments that will block the merging on the PR until the PR author fixes them. Makes it awesome to give to product, marketing or executives who can hold things in new and more difficult ways (I'm mostly kidding.)

> Vercel has "deployment checks" in their UI. If you're following along with me, you don't need them as the GitHub integration picks those up automatically. This would be if you were not using that.

[red]: https://github.com/btholt/fullstack-next-wiki/pull/2
