# Next.js Image — Configuration Reference

Set these up once per project. Unlike component props, these live in `next.config.js` or are passed as special props when the default optimizer isn't the right tool.

## Contents

- [remotePatterns — Whitelisting External Images](#remotepatterns--whitelisting-external-images)
- [loader — Custom CDN Support](#loader--custom-cdn-support)
- [unoptimized — Private / Auth Images](#unoptimized--private--auth-images)
- [How They Relate](#how-they-relate)

---

## remotePatterns — Whitelisting External Images

By default, Next.js proxies external images through its own optimizer. As a security gate — to prevent your server being used as an open image proxy — it refuses to fetch from domains you haven't explicitly allowed:

```js
// next.config.js
module.exports = {
  images: {
    remotePatterns: [
      { hostname: 'res.cloudinary.com' },
      { hostname: 'images.unsplash.com' },
      // supports wildcards:
      { hostname: '**.example.com' },
    ],
  },
}
```

You only need this when Next.js is doing the optimizing. If you're using a custom `loader` prop, the browser fetches directly from your CDN — Next.js is not in the middle, so `remotePatterns` doesn't apply.

| Scenario | Need `remotePatterns`? |
|---|---|
| External image, default optimizer | ✅ Yes |
| Custom `loader` prop per `<Image>` | ❌ No |
| Global custom loader in config | ✅ Yes (different config key) |
| Local `/public` images | ❌ No |

---

## loader — Custom CDN Support

### The problem with just passing a CDN URL

If you pass a Cloudinary URL as `src`, Next.js acts as a middleman:

```
Browser → /_next/image?url=https://res.cloudinary.com/...&w=800 → Next.js fetches + resizes → response
```

Cloudinary already does its own resizing — so you're double-optimizing, wasting server resources on something your CDN handles better.

### What loader does

It skips Next.js's optimizer and lets the CDN build its own URL directly:

```jsx
const cloudinaryLoader = ({ src, width, quality }) => {
  return `https://res.cloudinary.com/demo/image/upload/w_${width},q_${quality}/${src}`
}

<Image
  loader={cloudinaryLoader}
  src="profile.jpg"
  width={500}
  height={300}
/>
```

Now when Next.js needs a `500px` version, it builds the right Cloudinary URL and sends the browser there directly — no proxy, no double work.

| | Default (no loader) | Custom `loader` |
|---|---|---|
| Who optimizes? | Next.js (middleman) | Your CDN directly |
| Extra server cost? | ✅ Yes | ❌ No |
| CDN-specific features used? | ❌ Ignored | ✅ Fully used |
| Needs `remotePatterns`? | ✅ Yes | ❌ No |

> Think of it like: you *could* ask a friend to order pizza for you, or you could just call the restaurant directly. The `loader` is calling directly.

### Global loader (applies to every `<Image>`)

If you want the custom loader project-wide instead of passing it per component, set it in config:

```js
// next.config.js
module.exports = {
  images: {
    loader: 'cloudinary',
    path: 'https://res.cloudinary.com/your-account',
  },
}
```

---

## unoptimized — Private / Auth Images

When Next.js optimizes an image it fetches it server-side — and strips auth headers in the process. If your image is behind a login wall, that anonymous fetch fails:

```jsx
// Next.js optimizer fetches this without the user's cookies/tokens → 401
<Image src="https://private-cdn.com/secret-photo.jpg" width={400} height={300} />
```

`unoptimized` tells Next.js to step aside entirely — the browser fetches the image directly with its own credentials:

```jsx
<Image
  src="https://private-cdn.com/secret-photo.jpg"
  unoptimized
  width={400}
  height={300}
/>
```

| | Default (optimized) | `unoptimized` |
|---|---|---|
| Resizing / compression | ✅ Yes | ❌ No |
| Auth headers passed | ❌ No | ✅ Yes |
| Good for | Public images | Private / authenticated images |

You can also set `unoptimized: true` globally in `next.config.js` if you want to opt the entire project out of optimization (useful when deploying to environments without a Node.js server):

```js
module.exports = {
  images: {
    unoptimized: true,
  },
}
```

---

## How They Relate

All three are answers to the same question: **"who is responsible for fetching and serving this image?"**

```
Default
  Browser → Next.js optimizer → fetches source → resizes → serves back
  Requires: remotePatterns for external sources

loader
  Browser → CDN directly (Next.js just builds the URL)
  Requires: nothing in next.config.js

unoptimized
  Browser → source directly (Next.js steps aside entirely)
  Requires: nothing, but you lose all optimization
```

Use `remotePatterns` when you're happy with Next.js as the optimizer and just need to unlock external domains. Use `loader` when you have your own CDN that should do the work. Use `unoptimized` when optimization itself is the problem (auth, unsupported environments, already-optimized sources).