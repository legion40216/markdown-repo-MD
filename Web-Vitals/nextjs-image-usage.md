# Next.js `<Image>` — Usage Reference

## Contents

- [Why This Exists — and What next/image Does For You](#why-this-exists--and-what-nextimage-does-for-you)
- [Where Your Image Lives — and How to Pass src](#where-your-image-lives--and-how-to-pass-src)
- [fill vs width/height](#fill-vs-widthheight)
- [The sizes Prop](#the-sizes-prop)
- [Blur Placeholder Deep Dive](#blur-placeholder-deep-dive)
- [Placeholder Strategies](#placeholder-strategies)
- [priority / preload (Next.js 16+)](#priority--preload-nextjs-16)
- [Lazy Loading — What "Near Viewport" Means](#lazy-loading--what-near-viewport-means)
- [When to Use img Instead](#when-to-use-img-instead)
- [Debugging in DevTools](#debugging-in-devtools)
- [What Actually Matters — Priority Order](#what-actually-matters--priority-order)

---

## Why This Exists — and What next/image Does For You

When a browser assembles a page:

1. HTML is parsed → DOM is built
2. Browser sees `<img src="...">` → fires a **separate HTTP request**
3. Image bytes stream in → browser decodes → paints pixels
4. Until decoded: the space is empty, or you show a **placeholder**

This is why you see layout shifts and blank spaces — the browser doesn't know image dimensions until it downloads them. Every next/image feature exists to paper over this one original oversight. Two separate problems, two separate fixes:

**Layout shift** — when a browser parses HTML, it sees `<img src="/cat.jpg">` and has
no idea if that image is 100px or 4000px wide. It renders the layout with zero height
reserved, so when the image finally downloads the page jumps — that's the layout shift.

Fixed by always providing `width` and `height` so the browser can reserve the space upfront:

```html
<img src="/cat.jpg" width="800" height="600" alt="A cat" />
```

This tells the browser the aspect ratio before download, so it holds the slot open
immediately. `srcset` and `sizes` do NOT fix this — they only control which file gets
downloaded for the viewport. For dynamic images where dimensions aren't known ahead of
time, use a sized wrapper with `fill` instead.

**Wrong resolution — `srcset`** — one image file served to every screen size wastes
bandwidth on mobile. A phone user would download a 1200px image just to display it at
400px. The `w` descriptor gives the browser options:

```html
srcset="/cat-400.jpg 400w, /cat-800.jpg 800w, /cat-1200.jpg 1200w"
```

`sizes` then tells the browser how large the image will actually render on screen, so it
can pick the right file. Without it, the browser guesses — usually downloading too large.
The dedicated [sizes section](#the-sizes-prop) below covers this in full.

`<Image>` automates all of this behind a single component:

- **Auto-generates `srcset`** via its image optimization API (`/_next/image?url=...&w=800&q=75`)
- **Lazy loads** by default (only fetches when near viewport)
- **Prevents layout shift** by reserving space
- **Converts to WebP/AVIF** automatically
- **Caches** optimized versions on the server

---

## Where Your Image Lives — and How to Pass src

Before writing `src`, you need two things: where the file lives, and how you reference it. These are the same decision.

### src/assets — static import, bundled at build time

```jsx
import hero from '@/assets/hero.jpg'

<Image src={hero} alt="Hero" />
// width, height, and blurDataURL inferred automatically
```

You get for free: width/height inference, blur placeholder, content hashing, build-time error if the file is missing. Use for logos, hero images, decorative graphics — anything that's part of the codebase and known at build time.

### public/ — path string, served directly by URL

```jsx
<Image src="/assets/featured.jpg" alt="Featured" width={800} height={600} />
// width and height required — Next.js can't infer them
```

No automatic blur, no width/height inference — you provide those yourself. Use for favicons, PDFs, OG images, and anything user-uploaded or CMS-managed.

The reason `public/` exists: imports must be known at build time. Path strings can be dynamic:

```jsx
// ❌ impossible — imports must be known at build time
import image from `/products/${id}.jpg`

// ✅ works — path string can be dynamic
<Image src={`/products/${id}.jpg`} width={400} height={400} />
```

This is why `public/` is the only option for databases, CMS content, e-commerce, user uploads, and anything generated at runtime.

### Quick rule

| | `src/assets` | `public/` |
|---|---|---|
| Access via | `import image from '...'` | `src="/path/file.jpg"` |
| `width` / `height` | ✅ Auto-inferred | ❌ Required manually |
| Blur placeholder | ✅ Auto | ❌ Manual `blurDataURL` |
| Dynamic paths | ❌ No | ✅ Yes |
| Build-time check | ✅ Errors if missing | ❌ Fails silently at runtime |
| Good for | UI, logos, decorative images | CMS, uploads, favicons, PDFs, OG images |

**Rule of thumb:** static import for assets you know at build time. Path string when the URL is dynamic or comes from a database/API.

---

## fill vs width/height

**Use `width` + `height` when you know the dimensions:**

```jsx
<Image
  src="/avatar.png"
  alt="User avatar"
  width={64}
  height={64}
  className="rounded-full"
/>
```

**Use `fill` when you don't know dimensions** (CMS images, user uploads, dynamic content):

```jsx
// Parent MUST be position:relative with explicit dimensions
<div className="relative aspect-square w-full">
  <Image
    src={product.image}
    alt={product.name}
    fill
    className="object-cover"
    sizes="(max-width: 768px) 100vw, 50vw"
  />
</div>
```

> `fill` + `aspect-ratio` is essentially a 2009 CSS padding-top hack (`padding-top: 56.25%`) — Next.js just packages it cleanly.

### Does Next.js still optimize fixed width/height images?

Yes. Even with fixed numbers, Next.js still runs the image through its optimizer and generates a `srcset` — it uses your numbers as the target size:

```
// You write width={64} height={64}
// Next.js generates:
srcset="/_next/image?url=avatar.png&w=64   64w,
        /_next/image?url=avatar.png&w=128  128w"  ← retina (2× pixel density)
```

It serves `128w` on a retina display and `64w` on a normal screen — automatic, nothing extra needed.

### Do you need `sizes` with fixed width/height?

No. Because you've already told the browser exactly how big the image will render — it doesn't need to guess. `sizes` is only needed with `fill`, where the browser has no idea how wide the container is.

```jsx
// Fixed — browser knows it's always 64px, sizes not needed
<Image src="/avatar.png" width={64} height={64} />

// fill — browser doesn't know the container width, sizes required
<div className="relative w-full h-60">
  <Image src={product.image} fill sizes="(max-width: 640px) 50vw, 25vw" />
</div>
```

**Simple rule:** wrote `width` and `height` as fixed numbers → skip `sizes`. Wrote `fill` → always add `sizes`.

---

## The sizes Prop

You're telling the **browser** how wide the image will actually render at each screen size, so it picks the right resolution from the generated `srcset`.

```jsx
<Image
  src={src}
  alt={alt}
  fill
  sizes="(max-width: 640px) 100vw,   // full width on mobile
         (max-width: 1024px) 50vw,   // half width on tablet
         33vw"                        // third width on desktop
/>
```

The browser reads left to right and stops at the first match. Without `sizes`, it assumes `100vw` and downloads the largest image for every slot — even a 300px card.

**With `sizes`**, the browser picks the nearest match from the generated `srcset`:

```
You say 33vw on a 1400px screen → ~466px needed
Next.js srcset has 640w as nearest size up
Browser downloads 640px image, renders at 466px → fine, minimal waste
```

Rough `33vw / 50vw / 100vw` is good enough. Exact `calc()` math isn't necessary — the browser picks the nearest srcset entry anyway.

---

## Blur Placeholder Deep Dive

### What `blurDataURL` actually is

A tiny image (ideally 10px or smaller) encoded as text directly in the HTML — no separate HTTP request:

```
data:image/png;base64,iVBORw0KGgoAAAANS...
```

- `data:` — not a URL, it's raw data already in the HTML
- `image/png` — the format
- `base64,...` — the actual image bytes encoded as text

The browser renders it instantly (it arrives with the HTML itself), then crossfades to the real image on load.

### Which approach to use

| Situation | Approach |
|---|---|
| Statically imported images | `placeholder="blur"` — automatic, zero work |
| Remote images, just need a color | Hardcoded 1×1 `blurDataURL` from png-pixel.com |
| Remote images, real blur match | Server Component + Plaiceholder library |
| Client Component, dynamic | `bg-gray-100` on parent + `onLoad` fade — simplest |

### Dynamic blur with Plaiceholder

```ts
// lib/getBase64.ts
import { getPlaiceholder } from "plaiceholder";

export default async function getBase64(imageUrl: string) {
  try {
    const buffer = await fetch(imageUrl).then(res => res.arrayBuffer());
    const { base64 } = await getPlaiceholder(Buffer.from(buffer));
    return base64;
  } catch (e) {
    return undefined;
  }
}
```

```tsx
// Server Component usage
export default async function DynamicBlurImage({ src, alt }) {
  const base64 = await getBase64(src);
  return (
    <Image src={src} alt={alt} fill placeholder="blur" blurDataURL={base64} />
  );
}
```

> Costs a server-side fetch per image at request time. For a blog, a single hardcoded `FALLBACK_BLUR` constant is completely fine — the placeholder's job is just to stop a white flash, not match every image perfectly.

### CMS platforms often provide blur for free

```jsx
// Sanity
blurDataURL={post.coverImage.asset.metadata.lqip}

// Contentful
blurDataURL={`${imageUrl}?w=10&q=10`}
```

---

## Placeholder Strategies

Both solutions solve the same "blank box" problem with different aesthetics:

| | `placeholder="blur"` | Shimmer |
|---|---|---|
| **Feels like** | Image coming into focus | Content loading |
| **Best for** | Blogs, editorial, photography | E-commerce feeds, dashboards |
| **Customizable** | ❌ Blur only | ✅ Anything |
| **Requires image data** | ✅ Yes | ❌ No |

**Shimmer boilerplate:**

```jsx
"use client";
import Image from "next/image";
import { useState } from "react";

export default function ProductImage({ src, alt, sizes }) {
  const [loaded, setLoaded] = useState(false);

  return (
    <div className="relative w-full aspect-square overflow-hidden rounded-xl bg-gray-100">
      {!loaded && (
        <div className="absolute inset-0 animate-pulse bg-gray-200">
          <div className="absolute inset-0 -translate-x-full animate-[shimmer_1.5s_infinite] bg-gradient-to-r from-transparent via-white/60 to-transparent" />
        </div>
      )}
      <Image
        src={src || "/placeholder.png"}
        alt={alt}
        fill
        sizes={sizes || "(max-width: 768px) 100vw, 50vw"}
        className={`object-cover transition-opacity duration-500 ${
          loaded ? "opacity-100" : "opacity-0"
        }`}
        onLoad={() => setLoaded(true)}
      />
    </div>
  );
}
```

**Blur placeholder boilerplate:**

`FALLBACK_BLUR` is a 1×1 grey pixel as a base64 data URL — define it once and reuse everywhere. It arrives in the HTML itself, so it paints instantly with no extra request. Want a custom brand color? Go to [png-pixel.com](https://png-pixel.com), pick any color, and replace the value.

```ts
// utils/constants.ts
export const FALLBACK_BLUR =
  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg==";
```

```jsx
import { FALLBACK_BLUR } from "@/utils/constants";

<div className="relative aspect-[4/3] w-full overflow-hidden rounded-xl bg-gray-100">
  <Image
    src={src || "/placeholder.png"}
    alt={alt}
    fill
    preload={isAboveFold}
    sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"
    placeholder="blur"
    blurDataURL={blurDataURL || FALLBACK_BLUR}
    className="object-cover transition-opacity duration-300"
    onLoad={() => setLoaded(true)}
  />
</div>
```

---

## priority / preload (Next.js 16+)

> ⚠️ `priority` is deprecated in Next.js 16. Use `preload` instead.

```jsx
// ❌ Deprecated
<Image src={src} alt={alt} priority />

// ✅ Current
<Image src={src} alt={alt} preload={true} />

// Also valid
<Image src={src} alt={alt} loading="eager" fetchPriority="high" />
```

Use `preload` only on **above-the-fold** images. It disables lazy loading and emits a `<link rel="preload">` in the `<head>`, telling the browser to start fetching as early as possible.

---

## Lazy Loading — What "Near Viewport" Means

The browser doesn't wait until the image hits the exact screen edge — it starts fetching slightly before you'd see it:

```
┌─────────────────────┐  ← Top of screen
│   VISIBLE AREA      │
└─────────────────────┘  ← Bottom of screen
         │
         │  ~~ buffer zone ~~
         │  (roughly 1–3 screen heights,
         │   adjusts for connection speed)
         │
┌─────────────────────┐
│   image is HERE     │  ← Fetch starts NOW, not when you scroll here
└─────────────────────┘
```

The buffer makes scrolling feel instant. Slow 3G = bigger buffer, fetches earlier. Controlled manually via `rootMargin` in the Intersection Observer API — `next/image` handles this for you.

---

## When to Use `<img>` Instead

| Use `<Image>` | Use `<img>` |
|---|---|
| Any real content image | Tiny icons / SVGs (overkill) |
| Unknown / dynamic sources | Images already optimized by an external CDN |
| Performance matters | Email templates / non-Next environments |
| CMS / user-uploaded content | Base64 inline images |

---

## Debugging in DevTools

**Network tab → Img filter:**

```
/_next/image?url=your-image.jpg&w=640&q=75
                                  ↑
                            width Next.js chose
```

Resize the browser, hard refresh — watch `w=` change.

**Right-click → Inspect:**

```html
<img
  srcset="
    /_next/image?url=...&w=640   640w,
    /_next/image?url=...&w=750   750w,
    /_next/image?url=...&w=1080  1080w
  "
  sizes="(max-width: 640px) 50vw, 25vw"
  src="/_next/image?url=...&w=750"  ← which one the browser picked right now
/>
```

**What good looks like:**

```
Mobile 390px screen, image is 50vw → ~195px needed
✅ w=256 downloaded   — small, close to what's needed
❌ w=1920 downloaded  — sizes prop is wrong or missing
```

A properly optimized card image on mobile should be **20–80 KB**. 400 KB+ for a small card = something is wrong.

---

## What Actually Matters — Priority Order

```
1. sizes prop              ← biggest perf impact — saves downloading huge images
2. fill + aspect ratio     ← prevents layout shift
3. preload (above-fold)    ← hero images load fast
4. placeholder blur        ← nice polish, not critical
5. onLoad fade             ← pure aesthetics
```

The `sizes` prop has more real-world performance impact than anything in the placeholder stack. Nail that first.