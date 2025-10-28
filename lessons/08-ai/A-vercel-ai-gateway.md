---
title: "Vercel AI Gateway"
---

On the home page we want nice, tidy summaries of all of our docs. We could hire an editorial team to do it, make our own authors write their own, or just out the first two sentences of a document. All of these either make a bad summary or incur a lot of work. Let's instead use AI. One thing that current AI does extremely well (perhaps the best) is take long docs and summarize them done to salient points. We're going to do that now.

> As far as I know, there's no totally free way to do AI that doesn't require a credit card. If you need something like that, best I can suggest is set up Ollama with a very small model. I can suggest maybe like gemma3:270m or qwen3:0.6b. [Click here if you need Ollama instructions][ollama]. Just note that this will only work locally and won't deploy when we go later to deploy.

So let's go use Vercel AI Gateway. You could easily here either directly use OpenAI or Anthropic if you have credits with them (and you can still use the AI SDK, we'll see later) or you could use OpenRouter too which what I've used historically. This was my first time with their AI Gateway product and it works great!

Go to your Vercel dashboard, click on the AI Gateway tab. Click on Create API Key and give it a name. Copy this key into your .env file.

```bash
AI_GATEWAY_API_KEY="<your key>"
```

Next you probably need to stick a credit card in here to get the $5 free worth of credit. I know that's annoying but these free tiers get abused so much that I understand why (we have similar issue with Neon's free tier.) You could probably go buy a Visa $5 gift card to get started or use something like Privacy.com to set up one that has hard limits on it. Up to you. Or you could use Ollama locally but it wouldn't work deployed.

Head to your project and `npm i ai` to get the Vercel AI SDK. This is probably one of the biggest selling points to me for using Vercel's AI Gateway - the SDK is really nice to use and allows you to easily switch between OpenAI, Anthropic, Google, etc. without having to modify any code.

[ollama]: https://docs.ollama.com/quickstart
