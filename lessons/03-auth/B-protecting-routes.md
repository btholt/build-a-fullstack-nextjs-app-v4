We need to protect our edit page. Right now any anonymous user can load that page which is not what we want - if you are logged out, we want you to log in first before we let you edit.

> We haven't set up the database yet so we'll only just be differentiating between logged in and logged out. We'll revisit when we add the database to restrict edits to your articles only or allow admins to edit any article.

There are three ways to protect a route with Stack Auth (and I'd say it's pretty generally true for all of Next.js): client-side, server-side, or middleware.

## Client Side

A client-side redirect will look like `useUser({ or: 'redirect' });`. This tells Next.js "hey, if there's no user here, just redirect them to log in". You can also redirect them anywhere, but this is the most common thing you'd do. This works just fine in many cases and is a good user experience as the client can immediately redirect a user without a roundtrip to the server.

What's wrong with client side redirects? The code for these pages is all still sent to the browser, regardless if a user can access it or not. If the page contains sensitive data or shows off hidden endpoints that you may not want to leak or anything that truly is sensitive, you don't want to do this as any attacker could load up your JS and find whatever info you were trying to protect.

In our case it's totally fine - we're not trying to hide the editor from anyone, we just don't don't want to show to anyone who wouldn't be able to use it. We can absolutely use a client side redirect here.

## Server Side

Likewise a server side redirect will look like `await stackServerApp.getUser({ or: 'redirect' });`. This will make sure it all happens on the server and anything inside the component can be guaranteed not to be sent to the user unless they pass the login test. You'll find yourself doing this for React Server Components.

Something similar will likely also be done for API routes should you choose to implement those. In that case you'll do `await stackServerApp.getUser()` and then do a Next.js redirect if the user object doesn't exist.

Again, it's important to note, all of these are plenty secure - no one is going to be able to impersonate someone else, but it's just a matter if you care that the components are being sent down and not used for a user that doesn't pass the authorization.

## Middleware

The code for this will look like

```typescript
// from the stack auth docs
export async function middleware(request: NextRequest) {
  const user = await stackServerApp.getUser();
  if (!user) {
    return NextResponse.redirect(new URL("/handler/sign-in", request.url));
  }
  return NextResponse.next();
}

export const config = {
  // You can add your own route protection logic here
  // Make sure not to protect the root URL, as it would prevent users from accessing static Next.js files or Stack's /handler path
  matcher: "/protected/:path*",
};
```

This is what you'd do if you wanted to protect a whole block of routes. In the case of the example code above, you'd be gating access to `/protected/<anything>` so you wouldn't have to do it on each page. We won't do this today, but I wanted to let you know you don't have to do it page-by-page, you can do it in blocks too with middleware.

So let's go do it for our app!

In `/src/app/wiki/edit/[id]/page.tsx` put this

```typescript
// at top
import { stackServerApp } from "@/stack/server";

// under pulling the id out of params
await stackServerApp.getUser({ or: "redirect" });

// we'll uncomment this later when the articles have real IDs
// if (user.id !== id) {
//   stackServerApp.redirectToHome();
// }
```

This lets us either get the user object back from Stack Auth or, if it doesn't exist because they aren't logged in, redirects them to sign in or sign up. We put in the commented code because we can't yet check that because the articles don't have valid IDs, but when they do it will just send the user to the home page. In theory we should probably have a Forbidden page, but for now this is fine.

Okay, let's go do the new/page.tsx too

```typescript
// at top
import { stackServerApp } from "@/stack/server";

// replace function, add async
export default async function NewArticlePage() {
    await stackServerApp.getUser({ or: "redirect" });
    ...
}
```

The same! And there you go, we've protected our pages with server-side redirects! We did server-side because our pages were already React Server Components and there was no reason to convert them; client-side would have been just fine.

## Server Actions

We have some server actions to accept new and edited articles as well as image uploads. Just because those are server actions does not mean we can leave them unprotected. In reality they're just API endpoints too, even if they're not exposed as restful endpoints. Let's go protect those too (as it works mostly the same way too.)

For **both actions/upload.ts and actions/articles.ts** do the following

```typescript
// add import to the top
import { stackServerApp } from "@/stack/server";

// add this to the top of every action function
const user = stackServerApp.getUser();
if (!user) {
  throw new Error("❌ Unauthorized");
}
```

This still only checks that the user has logged in - we're still letting any user edit any article, but we'll get there soon. But that is auth! Congrats!
