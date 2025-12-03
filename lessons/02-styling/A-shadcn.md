---
title: "shadcn"
---

We are going to be using [shadcn ui][shadcn] for our design system. Let's dissect that a bit further.

- shadcn UI is based on [Radix][radix] (this is a good blog post on the difference). Radix is a component library that provides unstyled primitives that are well designed for accessibility and usability. shadcn takes these and adds a sane set of default styles to them.
- Both of these are in turn built on top of [Tailwind][tailwind]. Tailwind is a styling system that breaks every CSS value into its own CSS class, so instead of having stylesheets, you just directly apply the CSS to your React. If you've never used it before it can be abrasive but over time it's won over a critical mass of UI developers. When done well it's a delight to work in.

> Fun fact: [shadcn is a person][x]. He works at Vercel.

Essentially it's a opinionated set of styles and components. It's also a CLI that adds the components as you need them - it doesn't include everything at once which is nice.

Let's go ahead and initialize it.

```bash
npx shadcn@3.3.1 init
```

> Normally you should do `shadcn@latest` but we're going to be sticking to 3.3.1 for this course.

This will ask you to choose a tone. I think I went with slate? Feel free to choose your own base tone.

This should add some styles and make some modifications to your project. One thing to note is that this will include some global styles, but in and of itself shadcn is a component system. You need to individually add components. It does install a few dependencies like the icon library, some CSS helpers, and such. Don't worry about those too much - it's all for shadcn.

```bash
npx shadcn@3.3.1 add @shadcn/navigation-menu
npx shadcn@3.3.1 add @shadcn/button
```

Notice inside your app directory there is now a component/ui directory, and it has a navigation-menu.tsx and a button.tsx file in it. This is how shadcn works - it adds the code for the component to your library. This is cool because now it's _your_ component. Rather than trying to rebase and monkey patch a library, you can just directly edit the code. Long term this is more sustainable for using shadcn to craft your own design system instead of just a thin wrapper on top of something like Bootstrap.

Let's go ahead and make a navigation menu using our new component (we're not going to modify any of the shadcn components themselves today but you should feel free to, that's why the code is in your codebase!)

Create `nav-bar.tsx` in your components directory and put:

```typescript
import * as React from "react";
import Link from "next/link";
import {
  NavigationMenu,
  NavigationMenuList,
  NavigationMenuItem,
} from "@/components/ui/navigation-menu";
import { Button } from "@/components/ui/button";

export function NavBar() {
  return (
    <nav className="w-full border-b bg-white/80 backdrop-blur supports-[backdrop-filter]:bg-white/60 sticky top-0 z-50">
      <div className="container mx-auto flex h-16 items-center justify-between px-4">
        <div className="flex items-center gap-2">
          <Link
            href="/"
            className="font-bold text-xl tracking-tight text-gray-900"
          >
            Wikimasters
          </Link>
        </div>
        <NavigationMenu>
          <NavigationMenuList className="flex items-center gap-2">
            <NavigationMenuItem>
              <Button asChild variant="outline">
                <Link href="/signin">Sign In</Link>
              </Button>
            </NavigationMenuItem>
            <NavigationMenuItem>
              <Button asChild>
                <Link href="/signup">Sign Up</Link>
              </Button>
            </NavigationMenuItem>
          </NavigationMenuList>
        </NavigationMenu>
      </div>
    </nav>
  );
}
```

This is what working in Tailwind feels like (and thus Radix and shadcn, since they all use Tailwind). People think this feels gross, not using CSS and putting it all in the class, but here's the pitch. Everything we make in React is a component, and in theory we should compose all our pages of components that are glued together with bits of CSS. If you're following this pattern, all the components get self-contained CSS in their React components, and the bit of CSS you would write to glue pages together well ends up just being on the page itself via Tailwind classes. If you have something that _should_ share CSS with something else, then it should be a component.

In practice, I find this to be about 95% true - most things can just live in Tailwind classes with components. If you've ever maintained a large project before, you know deleting CSS is the hardest thing to do - it's so hard to tell what's used and what's not. When I worked at LinkedIn in 2015-ish, they had _2 megabytes_ of CSS, most of which was hand written. They had no idea how to fix it, and it's ultimately where the [CSS Blocks][blocks] project came from (which if it didn't inspire Tailwind, it certainly had similar goals.) We've tried so many ways to essentially get to this point - where code and styling are tightly linked in an obvious and non-footgun sort of way: [BEM][bem], [Atomic CSS][atomic], [styled-components][styled], [Emotion][emotion], and [CSS modules][modules]. [Sass][sass], [Less][less], and [Stylus][stylus] deserve mention too!

As you can see, we've been around the bend and many a heated discussion has been had on how best to style a project, and this just one of them (albeit the most popular at the moment.) I'm okay if you choose not to use Tailwind or these things on your projects, but it's what we're going to use today.

Okay, let's go add our nav bar to layout. Go to layout.tsx and add

```typescript
// at top
import { NavBar } from "@/components/ui/nav-bar";

// just inside <body>
<NavBar />;
```

Run your dev server with `npm run dev` and you should see your nav bar!

Let's do one more and add wiki card components to show the loop again. Go to [shadcn's docs][docs] and look at what's available. I see a Card component, let's use that.

```bash
npx shadcn@3.3.1 add @shadcn/card
```

Then create a wiki-card.tsx file in your ui component directory.

```typescript
import * as React from "react";
import Link from "next/link";
import {
  Card,
  CardHeader,
  CardTitle,
  CardDescription,
  CardContent,
  CardFooter,
} from "@/components/ui/card";

interface WikiCardProps {
  title: string;
  author: string;
  date: string;
  summary: string;
  href: string;
}

export function WikiCard({
  title,
  author,
  date,
  summary,
  href,
}: WikiCardProps) {
  return (
    <Card>
      <CardHeader className="pb-2">
        <div className="flex items-center gap-2 text-xs text-muted-foreground">
          <span>{author}</span>
          <span>•</span>
          <span>{date}</span>
        </div>
        <CardTitle className="text-lg">{title}</CardTitle>
      </CardHeader>
      <CardContent className="py-0">
        <CardDescription>{summary}</CardDescription>
      </CardContent>
      <CardFooter className="pt-2">
        <Link
          href={href}
          className="text-blue-600 hover:underline text-sm font-medium w-fit"
        >
          Read article &rarr;
        </Link>
      </CardFooter>
    </Card>
  );
}
```

Nothing too crazy here. Let's go redo our page.tsx to use it. (Feel free to copy/paste here.)

```typescript
import { NavBar } from "@/components/ui/nav-bar";
import { WikiCard } from "@/components/ui/wiki-card";

export default function Home() {
  return (
    <div>
      <NavBar />
      <main className="max-w-2xl mx-auto mt-10 flex flex-col gap-6">
        <WikiCard
          title="Complete Intro to React"
          author="Brian Holt"
          date="Sep 2025"
          summary="Learn React from the ground up with Brian Holt. Covers components, hooks, state, effects, and building modern UIs. Perfect for beginners and those wanting a solid foundation."
          href="https://frontendmasters.com/courses/complete-react-v9/"
        />
        <WikiCard
          title="Rust for TypeScript Developers"
          author="ThePrimeagen"
          date="Sep 2025"
          summary="ThePrimeagen teaches Rust to JavaScript/TypeScript devs. Dive into Rust's memory safety, ownership, and concurrency with fun, practical examples."
          href="https://frontendmasters.com/courses/rust-typescript/"
        />
        <WikiCard
          title="API Design & Node.js"
          author="Scott Moss"
          date="Sep 2025"
          summary="Scott Moss covers building robust APIs with Node.js. Learn REST, authentication, testing, and best practices for scalable backend services."
          href="https://frontendmasters.com/courses/api-design-nodejs/"
        />
        <WikiCard
          title="CSS Grid & Flexbox"
          author="Steve Kinney"
          date="Sep 2025"
          summary="Steve Kinney demystifies CSS Grid and Flexbox. Master layout techniques for responsive, modern web apps with hands-on demos and clear explanations."
          href="https://frontendmasters.com/courses/css-grid-flexbox-v2/"
        />
      </main>
    </div>
  );
}
```

That's it! That's the whole loop for managing shadcn, Tailwind, and Radix.

> 🏁 This is the [01-shadcn][checkpoint] checkpoint. Open that folder in the sample project repo to go to where we are as of right here.

[checkpoint]: https://github.com/btholt/fullstack-next-wiki/tree/main/01-shadcn
[shadcn]: https://ui.shadcn.com/
[radix]: https://workos.com/blog/what-is-the-difference-between-radix-and-shadcn-ui
[x]: https://x.com/shadcn
[blocks]: https://css-blocks.com/
[modules]: https://github.com/css-modules/css-modules
[bem]: https://getbem.com/
[styled]: https://styled-components.com/
[emotion]: https://emotion.sh/docs/introduction
[atomic]: https://acss-io.github.io/atomizer/
[sass]: https://sass-lang.com/
[less]: https://lesscss.org/
[stylus]: https://stylus-lang.com/
[docs]: https://ui.shadcn.com/docs/components
[tailwind]: https://tailwindcss.com/
