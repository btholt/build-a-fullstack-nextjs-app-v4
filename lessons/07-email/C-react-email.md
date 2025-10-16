Have you ever tried to write markup for emails? It's horrible. It's like living twenty-five+ years in the past. Even basic CSS is asking too much.

Enter React Email - they built a framework around writing normal React and it takes that and outputs markup for email clients that works. It's so much better than things ever used to be. So, so, so much better.

So let's modify our celebration email to use an email template with React Email.

Let's make a templates directory inside of the emails folder. In there, let's put a celebration-template.tsx and put this.

```typescript
type Props = {
  name?: string;
  pageviews: number;
  articleTitle?: string;
  articleUrl?: string;
};

const CelebrationTemplate = ({
  name,
  pageviews,
  articleTitle,
  articleUrl,
}: Props) => {
  return (
    <html>
      <body
        style={{
          backgroundColor: "#f8fafc",
          margin: 0,
          padding: 20,
          fontFamily:
            "Inter, ui-sans-serif, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial",
          color: "#0f172a",
        }}
      >
        <table
          width="100%"
          cellPadding={0}
          cellSpacing={0}
          role="presentation"
          style={{ maxWidth: 600, margin: "0 auto" }}
        >
          <tr>
            <td style={{ paddingBottom: 12 }}>
              <div style={{ textAlign: "left" }}>
                <strong style={{ fontSize: 18, color: "#0f172a" }}>
                  Wikimasters
                </strong>
              </div>
            </td>
          </tr>

          <tr>
            <td>
              <div
                style={{
                  background: "#ffffff",
                  borderRadius: 8,
                  padding: 24,
                  boxShadow: "0 1px 0 rgba(15,23,42,0.04)",
                }}
              >
                <h1
                  style={{
                    margin: "0 0 8px 0",
                    fontSize: 20,
                    lineHeight: "28px",
                  }}
                >
                  🎉 Nice work{name ? `, ${name}` : ""}!
                </h1>

                <p style={{ margin: "0 0 16px 0", color: "#334155" }}>
                  Your article{articleTitle ? ` "${articleTitle}"` : ""} just
                  hit <strong>{pageviews}</strong> views — that's a milestone.
                </p>

                {articleUrl ? (
                  <a
                    href={articleUrl}
                    style={{
                      display: "inline-block",
                      textDecoration: "none",
                      background: "#0ea5a4",
                      color: "white",
                      padding: "10px 14px",
                      borderRadius: 6,
                      fontWeight: 600,
                    }}
                  >
                    View article
                  </a>
                ) : null}

                <p style={{ marginTop: 18, color: "#94a3b8", fontSize: 13 }}>
                  Keep writing — you're helping other people learn. — The
                  Wikimasters team
                </p>
              </div>
            </td>
          </tr>

          <tr>
            <td style={{ paddingTop: 14 }}>
              <p style={{ margin: 0, color: "#94a3b8", fontSize: 12 }}>
                You’re receiving this email because you authored content on
                Wikimasters.
              </p>
            </td>
          </tr>
        </table>
      </body>
    </html>
  );
};

export default CelebrationTemplate;
```

- Pretty simple React using inline styles
- You can use the [react-email/tailwind][tw] if you feel like you need to be consistent. I found it to be less helpful than just inlining styles.
- Otherwise this is just simple React.

Let's go make it work. Rename `celebration-email.ts`'s file extension to `celebration-email.tsx` so it can use React in it. Now modify it.

```typescript
// at top
import CelebrationTemplate from "./templates/celebration-template";

const BASE_URL = "http://localhost:3000";

// grab name and title
const response = await db.select({
  email: usersSync.email,
  id: usersSync.id,
  title: articles.title,
  name: usersSync.name,
});

// grab title and name too
const { email, id, title, name } = response[0];

const emailRes = await resend.emails.send({
  from: "Wikimasters <noreply@mail.holt.courses>", // replace with your domain when ready
  to: email,
  subject: `✨ Your article got ${pageviews} views! ✨`,
  react: (
    <CelebrationTemplate
      articleTitle={title}
      articleUrl={`${BASE_URL}/wiki/${articleId}`}
      name={name ?? "Friend"}
      pageviews={pageviews}
    />
  ),
});
```

- Normally I'd be a bit more useful about the BASE_URL, but for here it's fine for the email
- Otherwise it's jsut getting the right data to render.
- These emails look great! And we don't have to write painful email templates.

Congrats! Adding email to a modern app is so easy. Adding something like Twilio for texting isn't super hard either, it'll feel very similar.

[react-email]: https://react.email/
[tw]: https://www.npmjs.com/package/@react-email/tailwind
