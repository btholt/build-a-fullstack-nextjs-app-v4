Let's hop into sending emails. Not the most exciting part of your stack but very important nonetheless. It's your way of being able to reach people when they're not actively using your app.

Email used to be so hard. As someone that had to work with email servers in-house, it is a level of complexity that you just never want to deal with. If you've never had to deal with reputation or deliverability of email, count your lucky stars. AWS SES (simple email service) was the first time I got to use something that allowed me to not worry about all that and it was a revelation. Then Sendgrid came along (now apart of Twilio) and made it that much easier.

Then came Resend - it made sending email as easy invoking a function and not much more. It's so easy and nice to use. Huge recommendation from me, I love the product. I love any product that can turn a negative experience into a positive one.

So let's go set it up. Head to [resend.com][resend] and sign up. When you're done signing up, grab your API key, and put it in your .env as `RESEND_API_KEY`.

Lastly go install

```bash
npm i resend @react-email/render
```

> Resend will only let you send emails to yourself until you set up a custom domain. You don't need to do that for our proof of concept, just be aware. I've set mine up so I can demo it for you, but it takes 15 minutes or so to set it up so I don't expect you to do it.

[resend]: https://resend.com
