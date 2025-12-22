Since this is a fullstack enterprise Next.js course, I wanted to impart some knowledge that I've earned by being involved in maintaining large Tailwind projects. It's generally the same principles as doing CSS, just flavored with having to do it with your React too.

## Take advantage of Tailwind's theming

Tailwind's theme system is built around theme variables - special CSS variables that define your design tokens (colors, fonts, spacing, etc.) and automatically generate corresponding utility classes. Instead of being stuck with predefined utilities, you can customize your design system by defining theme variables using the @theme directive.

For example, adding `--color-mint-500: oklch(0.72 0.11 178)` to your theme automatically creates utilities like `bg-mint-500`, `text-mint-500`, and `border-mint-500` throughout your project. The system is organized into namespaces like `--color-*` for colors, `--font-*` for font families, `--spacing-*` for sizing, and `--breakpoint-*` for responsive variants. You can extend the default theme by adding new variables, override existing ones, or completely replace entire sections. This approach gives you a consistent design system where your custom values work seamlessly with Tailwind's utility-first methodology, and your theme variables are also available as regular CSS variables for use in custom CSS or inline styles.

I'm summarizing what's already very well written [in the Tailwind docs][tw] - please read through it as it's the difference between working in a good Tailwind codebase and a bad one.

In essence, I'm asking you to create and stick to design tokens - have standard colors, paddings, margins, etc. and name them. Otherwise you'll end up with 100s of ways to do the same thing and it's a mess.

## Make variants

Tailwind has the ability to create variants. Buttons are the best example of this - you'll have a CTA button, a normal button, a disabled button, a secondary button, etc. Rather than do something weird in the code or make a new component, just use Tailwind's built-in `@variant` directive to define what minor changes happen to each variant.

## cva is a godsend

[cva](https://cva.style), short for class-variance-authority (which I don't think is a reference to the TV show Loki but I can't help but think of it every time) is a library that helps manage variants. Using it, you can make variants that are conditionally applied based on what type of thing it is. Let's look at an example

```javascript
const button = cva("font-semibold border rounded", {
  variants: {
    intent: {
      primary: "bg-blue-500 text-white",
      secondary: "bg-gray-200 text-gray-800",
    },
    size: {
      small: "text-sm py-1 px-2",
      large: "text-lg py-3 px-6",
    },
  },
  defaultVariants: { intent: "primary", size: "small" },
});
```

Now we have two different ways to apply a button's style, size and intent, and we can apply different styles based on that. Even better, TypeScript will help enforce that the variants are correct and possible, making it _super_ useful to use. Highly recommend, it helps tame Tailwind in a way that makes a project scale.

## `@apply` is an anti-pattern

If you're unfamiliar with the `@apply` directive, it allows you to author CSS using Tailwind classes. Essentially you can make CSS classes using Tailwind. If find yourself doing this a lot, you're just doing CSS with extra steps, and it leads to the same problems as normal CSS and diminishes a lot of what makes Tailwind great. When you mix the two together, you end up with not having a single source of truth, and at that point you should just pick a system and stick with it.

Am I saying _never_ use @apply? Almost. Like, one step away from that. Treat it like `!important` - there are rare cases where it does make sense to use it, but otherwise you're just creating problems for yourself.

[tw]: https://tailwindcss.com/docs/theme
