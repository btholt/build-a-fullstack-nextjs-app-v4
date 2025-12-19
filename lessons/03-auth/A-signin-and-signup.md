> 🚨 The version of Neon Auth taught in this course is deprecated. This version of Neon Auth wrapped another service, Stack Auth. Luckily, you can still sign up directly for Stack Auth, but the database sync'ing that it did no longer works. I've modified the course notes to use Stack Auth. We will have to add several things that the videos don't have, but we've kept it fairly minimal so that the deviations from the videos is as small as possible. 🚨
> There is a new version of Neon Auth that uses Better Auth, and you can use that if you please, but it works slightly different and uses a different SDK. But otherwise it's pretty similar!

We are going to do now do auth using Neon Auth, which under the hood is the same as Stack Auth - they're just neatly linked if you use Neon and Neon Auth together.

> We're going to skip using Neon Auth here and use Stack Auth

- Head to Neon.com
- Sign up for an account
- Create a new project - use the most recent version of Postgres and it doesn't matter what cloud or region you choose
- Click the "Connect" button and copy the connection strings and paste it into a `.env` file at the root of your project.

> Now go to [stack-auth.com](https://stack-auth.com), sign up, and grab the environment keys.

- Create a new project on Stack Auth (fem-wikimasters)
- Click `Project Keys` in the sidebar menu
- Click `Create Project Keys` to generate keys for your project
- Copy the `Next.js` keys into your `.env` file

Your new .env file should look like

```
# Stack Auth environment variables for Next.js
NEXT_PUBLIC_STACK_PROJECT_ID='your project id'
NEXT_PUBLIC_STACK_PUBLISHABLE_CLIENT_KEY='your publishable key'
STACK_SECRET_SERVER_KEY='your secret key'

# Neon Database owner connection string
DATABASE_URL='your postgres connection string'
```

Now that we have that (and as a bonus we're ready to go for our database too because we got the connection string) we can start adding Neon Auth to our project.

Now go to the root of your project and run

```bash
npx @stackframe/init-stack@latest --no-browser
```

This should initiate your project and add some auth files to it.

Now you should be able to click sign in or sign up from the top bar and actually sign in or sign up. Stack Auth gives a really nice UI that already fits with shadcn so we don't need to restyle it or anything (though you certainly could!)

That's it for intro'ing our sign in and sign out systems. Modern auth companies make this so easy now - it used to be so hard!

So let's make it that our sign in and sign out buttons show when the user is signed out, and the official UserButton shows from Stack Auth when the user is signed in.

In `src/app/components/nav/nav-bar` put this

```typescript
// at top
import { UserButton } from "@stackframe/stack";
import { stackServerApp } from "@/stack/server";

// inside function
const user = await stackServerApp.getUser();

// replace inside menu list
{
  user ? (
    <NavigationMenuItem>
      <UserButton />
    </NavigationMenuItem>
  ) : (
    <>
      <NavigationMenuItem>
        <Button asChild variant="outline">
          <Link href="/handler/sign-in">Sign In</Link>
        </Button>
      </NavigationMenuItem>
      <NavigationMenuItem>
        <Button asChild>
          <Link href="/handler/sign-up">Sign Up</Link>
        </Button>
      </NavigationMenuItem>
    </>
  )
}
```

Here we're checking to see if a user exists (which would only exist if they were logged in.) If it exists, show the UserButton from Stack Auth. If not, show our sign in and sign up buttons. Also notice we're not using the useUser hook - this is a stateful, client-side hook. Because this is a React Server Component, we need to use the server getUser function instead.

Try clicking into the account settings from the UserButton - Stack Auth ships a whole user management system so you don't have to!
