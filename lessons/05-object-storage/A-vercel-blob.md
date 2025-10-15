- Head to Vercel.com
- Sign up
- Click "Storage" tab
- Click Create Database
- Make a Blob bucket
- Choose a region and a name
- `npm i @vercel/blob`
- Copy snippet
- Click .env.local, copy your key, paste into your .env
- Get the base url from the settting page, put it in next.config.js

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [new URL("<your_vercel_blob_url>/**")],
  },
};

export default nextConfig;
```

- Add articles.ts helper to return imageUrl
- Add to server actions, both update and create
