We are going to now do all the fun DevOps parts at the end: we are going to deploy, we are going to do CI/CD, and we are going to do observability and analytics.

There are so many ways to skin this cat so we are going to keep it pretty focused - Vercel offers nearly all of these services built-in and it works really well with Neon and Upstash, so we're just going to lean on that. If I was just getting started, this is what I'd do - only reason I'd something like a different observability platform or analytics is if I outgrew the perfectly adequate offering on Vercel.

Go to GitHub, create a new repo, and push your project into that repo. We're going to use this to deploy. Once you've pushed your code to GitHub, go to Vercel.com and create a new project and connect your GitHub so we can add your repo to this project.

I took the liberty here of fixing some stuff up for deployment, namely adding a bunch of tests. This isn't a testing Next.js course and if you want to see my opinions on testing Next.js [check out my Intermediate React course][ir6] - I go into plenty depth there. So instead I just want you to see what to do with the tests once you have them. I've added some test specific code that you can look at if you want (for example it doesn't run the AI summaries in test because it'd waste a lot of tokens)

> 🏁 This is the [09-with-tests][checkpoint] checkpoint. Open that folder in the sample project repo to go to where we are as of right here. This is the final version of the project and what we'll use to deploy.

Another note - these days I mostly vibe code my tests - it's such a perfect use case for it.

In this repo, it'll now expect _two_ sets of variables, if you want to. For the purposes of this course, feel free to just re-use your prod infra for test, but you'll see how to set up preview environments _and_ prod environments.

Neon will automatically branch your database for you after we set up the integration so don't worry about those.

I personally set up a second Upstash, gave a fake Resend key for email, a second blob storage (which you can connect via the Storage tab but you do still need to add the `BLOB_BASE_URL` manually)

For Neon Auth, it won't work as-is. This is somewhat intentional - Neon Auth today just has one environment that doesn't branch, so your preview env would be hitting production, which you may or may not want. I'd say you probably don't want that.

The way to work around this is to have a second Neon project that is just a Neon Auth project and you'd regularly make a copy of your production Neon Auth to the project Neon Auth so it didn't fall out of sync (you'd probably anonymize it too) - this isn't ideal and we're working on a better situation at Neon.

Alternatively you could have your preview environments automatically add their redirect URLs on creation and then have a job to clean them up when they go away. Messy still too.

Or, and hear me out here, you make the devs do it manually for each preview environment that needs it. Annoying, yes, but it also makes the devs handle it and it's not that burdensome to do - it's a couple of clicks in the Neon UI. We're going to skip doing this.

Lastly I just add the AI_GATEWAY_KEY for all envs - it won't get used in test because I turned it off in test.

> 🚨 Be sure to not upload every step (like my repo is) - instead just upload the project you're working on.

We don't need to modify the build or anything because it all should just be default Next.js - Vercel is very much built for Next.js and vice-versa.

Once you've done this, your app should be in prod! If you gave Vercel all the permissions it needed, it should also automatically set up all the Vercel preview deployment stuff with no additional work.

Let's hook up everything we need now

## Neon

Let's make Neon work. Vercel and Neon have taken great care to make sure this works really well together so experience is as easy as

- Go to the project dashboard
- Go to Settings
- Go to Integrations
- Browse Marketplace
- Search for and Click on Neon. [You can also go straight there from here][neon]
- Click Install from the top right, click Link Existing Neon Account since that's what we did.
- Link your existing project

> You can install Neon via Vercel. The only real difference here is that you'll get one bill for Neon through Vercel. Otherwise it's the same product doing the same thing.

That's it! This will handle all secret management and branching for with regards to Vercel.

## Neon Auth

Neon Auth won't work until you set the correct callback URL. You'll take your base Vercel URL which be something like `https://test-wiki-smoky.vercel.app/` and add that to Neon Auth's "Trusted Domains" under the configuration tab.

For Neon Auth, it won't work as-is for the _preview environments_. This is somewhat intentional - Neon Auth today just has one environment that doesn't branch, so your preview env would be hitting production, which you may or may not want. I'd say you probably don't want that.

The way to work around this is to have a second Neon project that is just a Neon Auth project and you'd regularly make a copy of your production Neon Auth to the project Neon Auth so it didn't fall out of sync (you'd probably anonymize it too) - this isn't ideal and we're working on a better situation.

Alternatively you could have your preview environments automatically add their redirect URLs on creation and then have a job to clean them up when they go away. Messy still too.

Or, and hear me out here, you make the devs do it manually for each preview environment that needs it. Annoying, yes, but it also makes the devs handle it and it's not that burdensome to do - it's a couple of clicks in the Neon UI. We're going to skip doing this.

## Upstash

Feel free to link up Upstash here as well, works the same way. I don't really _use_ the integration that much - I think it has some novel behavior in Upstash's dashboard that can monitor different things. Really I just use it to make sure the environment variables stay in sync.

## Resend

Same as above - this will just create a new API key on the fly and you can track Vercel usage individually. Fine to do or not do, as long as you have a Resend key.

## Blob

Likewise here, via the Storage tab, you can create a link to your Vercel Storage account so that the UI links up nicely. Or you can just make sure the right variables are populated.

> For some of these, you may need to delete old keys that you put there first to set up the integrations. Vercel Blob is certainly that way (ask me how I know)

If all of your variables are set, you should be able to see your site working!

[ir6]: https://holt.fyi/intermediate-react
[checkpoint]: https://github.com/btholt/fullstack-next-wiki/tree/main/09-with-tests
[neon]: https://vercel.com/marketplace/neon
