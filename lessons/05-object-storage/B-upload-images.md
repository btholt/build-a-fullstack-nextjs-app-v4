Let's start with making the image uploads work. You can do this one of two ways with Vercel Blob (and many other storage solutions): directly from the client to Vercel, or from the client to your server to Vercel. The former skips the middleman and means your server doesn't have to know or care about image uploads which is nice. What happens is your server mints a temporary token that allows access to upload a file to Vercel and the clients send it there.

Since we have all the server actions already set up, it's really easy for us to just do it via the server. What's advantageous about this is we could do any post-processing we wanted on the server too - resize, make thumbnails, etc. in this code which we wouldn't be able to do on the client. Both approaches work, we just went with the simpler one given our setup.

In actions/upload.ts, put this

```typescript
// at top
import { put } from "@vercel/blob";

// replace mock code
try {
  const blob = await put(file.name, file, {
    access: "public",
    addRandomSuffix: true,
  });

  return {
    url: (blob as any).url ?? "",
    size: file.size,
    type: file.type,
    filename: (blob as any).pathname ?? file.name,
  };
} catch (err) {
  console.error("❌ Vercel Blob upload error:", err);
  throw new Error("Upload failed");
}
```

- Not too bad here. We upload an image to Vercel blob that we got via form data and get back an imageUrl that we save in the database.
- `access: "public"` is essential as we want anyone to able to see these images.
- `addRandomSuffix: true` is also important - if I upload `pic.jpg` and then you do with the same file name, it would overwrite it. But with this property set it's guaranteed to not collide.
- I already did all the validation that it's an image, not over 10MB, it's attached, etc. for you.

Last thing, we need to save these images to the database and then make sure we select them for showing as well.

In actions/articles.ts

```typescript
// add to createArticle and updateArticle
imageUrl: data.imageUrl ?? undefined,
```

`??` is the nullish operation - if the first thing is falsey then it gives you the second thing. In thise case if imageUrl is undefined, then it returns undefined. We could leave out the `?? undefined` but I feel like this is more clear.

In lib/data/articles

```typescript
// add to getArticleById select
imageUrl: articles.imageUrl,
```

Now give it shot and see that it works!

> 🏁 This is the [05-object-storage][checkpoint] checkpoint. Open that folder in the sample project repo to go to where we are as of right here.

[checkpoint]: https://github.com/btholt/fullstack-next-wiki/tree/main/05-object-storage
