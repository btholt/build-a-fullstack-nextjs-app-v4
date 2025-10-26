---
title: "Speed Insights, Analytics, and Observability"
---

Let's just go over a few tabs that I want you to look at. I'm not going to cover them too in-depth, just that you should care they're there and if this was a real app, you should really keep an eye on them

## Analytics

Who are your users? This tab aims to answer part of that. If you've ever looked at Google Analytics, this will look familiar - what pages are getting traffic, what the bounce rate you have (people coming to your site and "bouncing" right away, generally a bad thing), referers (did they get here via Google or something else), what browser they're using, etc.

They also have some cool features around feature flagging and custom events that are for pro users, take a look at those if you're interested.

Generally speaking, this is sufficient for new projects, but you probably will eventually needed more sophisticated tracking. But to get started with, it's great it's all in one place on the same bill.

I put this for you already, but you just need to include a little script to track these events. It's in layout.tsx, and I also put the Speed Insights script for you.

## Speed Insights

Think of this like an automated Google Lighthouse, where it tells you your basic web performance metrics and tracks them over time. Super useful for monitoring how well you're performing and track regressions with new pushes. Highly recommend turning this on for real projects you do - it's proven web performance does make a difference on customer behavior.

## Logs

Just like you have in your terminal when you run it locally, this is those logs. We could definitely do better formatting to be more searchable and useful from a prodution perspective, but for now this is great.

## Observability

Definitely came an eye on this - this will be how many requests are error'ing out, how much data you're using, and other stuff like that. If you look on the side bar you can track which external APIs you're calling (in our case, we call Upstash a lot) as well images, functions, and other things. They even (with a Pro account) let you set up alerts if stuff starts breaking. This is amazing because it's kind of annoying to set up otherwise.

## Firewall

Fairly new feature, and very cool. There's two things in here I really like: bot protection and attack mode.

Bot protection is if you have a site that you want snooping bots and LLMs to not use. It'll traffic analysis and from what I hear it works pretty well. Think Cloudflare Turnstile. I see this as being really useful for preventing automated sign ups and other things where you're giving things away where people would be incentived to steal it, like offering a free database that people can use 😅

Attack mode works like Cloudflare's as well. If you're the target of a DDoS attack, turn this on and Vercel will mitigate it for you. It will make your users have to likely do a Captcha or something before they land on your site, but a small price to pay versus just going down all together. Very cool they have it. There's other stuff here but I thought I'd highlight that.
