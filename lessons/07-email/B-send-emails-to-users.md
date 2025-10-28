We're going to send celebration emails every time your

Make a new directory in your src directory called `email`. In there put this index.ts

```typescript
import { Resend } from "resend";

const resend = new Resend(process.env.RESEND_API_KEY);

export default resend;
```

Now create in the same directory celebration-email.ts

```typescript
import resend from "@/email";
import db from "@/db";
import { usersSync } from "drizzle-orm/neon";
import { articles } from "@/db/schema";
import { eq } from "drizzle-orm";

export default async function sendCelebrationEmail(
  articleId: number,
  pageviews: number
) {
  const response = await db
    .select({
      email: usersSync.email,
      id: usersSync.id,
    })
    .from(articles)
    .leftJoin(usersSync, eq(articles.authorId, usersSync.id))
    .where(eq(articles.id, articleId));

  const { email, id } = response[0];
  if (!email) {
    console.log(
      `❌ skipping sending a celebration for getting ${pageviews} on article ${articleId}, could not find email`
    );
    return;
  }

  console.log(email);

  // OPTION 1: this only works if you've set up your own custom domain on Resend like I have
  const emailRes = await resend.emails.send({
    from: "Wikimasters <noreply@mail.holt.courses>", // should be your domain
    to: email,
    subject: `✨ You article got ${pageviews} views! ✨`,
    html: "<h1>Congrats!</h1><p>You're an amazing author!</p>",
  });

  // OPTION 2: If you haven't set up a custom domain (development/testing)
  // Uncomment this and comment out Option 1:
  //   const emailRes = await resend.emails.send({
  //     from: "Wikimasters <onboarding@resend.dev>", // I believe it only lets you send from Resend if you haven't set up your domain
  //     to: "<the email you signed up with>", // unless you set up your own domain, you can only email yourself
  //     subject: `✨ You article got ${pageviews} views! ✨`,
  //     html: "<h1>Congrats!</h1><p>You're an amazing author!</p>",
  //   });

  if (!emailRes.error) {
    console.log(
      `📧 sent ${id} a celebration for getting ${pageviews} on article ${articleId}`
    );
  } else {
    console.log(
      `❌ error sending ${id} a celebration for getting ${pageviews} on article ${articleId}`,
      emailRes.error
    );
  }
}
```

- Take note that you use the correct version - Resend restricts a lot until you set up your domain with Resend
- Otherwise this is pretty straightforward - more than email ever was!
- I'm in the habit of not logging emails to the logs for GDPR reasons, hence the ID instead of the email
- Obviously we'd want a more interesting email for this, but marketing isn't the point of this lesson.

Let's go make this work by adding it to pageviews.ts action.

```typescript
// at top
import sendCelebrationEmail from "@/email/celebration-email";
const milestones = [10, 50, 100, 1000, 10000];

// under newVal declaration
if (milestones.includes(newVal)) {
  sendCelebrationEmail(articleId, +newVal); // don't await so we don't block on sending the email, just send it
}
```

That's it!
