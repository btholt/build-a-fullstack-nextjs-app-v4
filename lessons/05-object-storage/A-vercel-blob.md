Next we're going to implement object storage. What is object storage? It's a place where you can upload files (or blobs, as many places call them, as it doesn't matter what the file is, it's just a blob of data.) There are tons of ways to do this, most commonly AWS S3, but we're going to go ahead and stick with Vercel since most of our project is based around Vercel, but you could use Cloudinary, S3, Azure Blob Storage, uploadthing, or any other number of services. Vercel Blob works just great.

What we want to do is allow users to attach an image to an article if they want to. So we're going to do that with Vercel Blob.

Head to Vercel.com and log in or sign up. From there, click on the "Storage" tab. Click "Create Database" (despite this not really being a database) and click "Blob" to create a new Vercel Blob bucket. Choose a region and a name.

> You may have noticed you can get Neon through Vercel as well - this is totally valid if you want to do it this way. Both Vercel and Neon and work hard to make this integration work great, and it's nice to get everything through one bill. I chose to do it this way because I wanted to teach database before I taught Vercel.

From here, go to you Vercel project and click the `.env.local` little tab to be able to copy your API key, and paste that into your .env file.

Next go to the settings page and copy the base URL. In your next.config.js file put this

```javascript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [new URL("<your_vercel_blob_url>/**")],
  },
};

export default nextConfig;
```

This allows you to use the `next/image` components with no additional config, it just works.

Go ahead and run this in your project to install the SDK

```bash
npm i @vercel/blob
```

Awesome. Now let's go write the code.
