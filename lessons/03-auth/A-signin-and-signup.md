We are going to do now do auth using Neon Auth, which under the hood is the same as Stack Auth - they're just neatly linked if you use Neon and Neon Auth together.

- Head to Neon.com
- Sign up for an account
- Create a new project - use the most recent version of Postgres and it doesn't matter what cloud or region you choose
- Click on the Neon Auth tab
- Create your Neon Auth integration
- Go to configuration, click generate keys
- Copy the snippet and paste it into a `.env` file at the root of your project.

Your new .env file should look like

```
# Neon Auth environment variables for Next.js
NEXT_PUBLIC_STACK_PROJECT_ID='your project id'
NEXT_PUBLIC_STACK_PUBLISHABLE_CLIENT_KEY='your publishable key'
STACK_SECRET_SERVER_KEY='your secret key'

# Database owner connection string
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
  );
}
```

Here we're checking to see if a user exists (which would only exist if they were logged in.) If it exists, show the UserButton from Stack Auth. If not, show our sign in and sign up buttons. Also notice we're not using the useUser hook - this is a stateful, client-side hook. Because this is a React Server Component, we need to use the server getUser function instead.

Try clicking into the account settings from the UserButton - Stack Auth ships a whole user management system so you don't have to!
