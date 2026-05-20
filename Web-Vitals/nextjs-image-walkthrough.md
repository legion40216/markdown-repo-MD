# Next.js `<Image>` — Progressive Optimization Walkthrough

You have a working component. This guide walks through every improvement you can make, in order of impact — stop whenever you're happy.

## Contents

- [Step 0 — The Starting Point](#step-0--the-starting-point)
- [Step 1 — Is Your Image Local or Remote?](#step-1--is-your-image-local-or-remote)
- [Step 2 — Add sizes (Most Important)](#step-2--add-sizes-most-important)
- [Step 3 — Add a Placeholder](#step-3--add-a-placeholder)
- [Step 4 — preload for Above-the-Fold Images](#step-4--preload-for-above-the-fold-images)
- [Step 5 — Blur at Scale (Gallery or Product Grid)](#step-5--blur-at-scale-gallery-or-product-grid)
- [Where to Stop](#where-to-stop)

---

## Step 0 — The Starting Point

A basic `fill` image with no extras is already valid — it won't break anything:

```jsx
<div className="relative w-full h-60">
  <Image
    src={src}
    alt={alt}
    fill
    className="object-cover"
  />
</div>
```

What it's missing: `sizes` (biggest issue), a placeholder (polish), and `preload` for above-fold images. Add them in order.

---

## Step 1 — Is Your Image Local or Remote?

This changes what's available to you.

**Local image** (lives in `/public` or imported directly) — use a static import and get `width`, `height`, and blur for free:

```jsx
import avatar from './avatar.png'

// Next.js infers width, height, and blurDataURL automatically
<Image src={avatar} alt="Avatar" className="rounded-full" />
```

**Known fixed size** (avatar, logo, icon) — just pass dimensions:

```jsx
<Image src="/avatar.png" alt="Avatar" width={64} height={64} className="rounded-full" />
```

**Unknown size / dynamic / remote image** (anything from a CMS, API, or user upload) — use `fill` with a positioned parent:

```jsx
<div className="relative w-full h-60">   {/* or aspect-square, aspect-video, etc. */}
  <Image
    src={src}
    alt={alt}
    fill
    className="object-cover"
  />
</div>
```

> If your image URL comes from a database, CMS, or API — it's remote. You're in the `fill` path.

---

## Step 2 — Add `sizes` (Most Important)

Without `sizes`, the browser assumes the image is full screen width and downloads a massive file even for a small card. `sizes` is how you tell it the truth.

The value you write depends on how wide your image actually renders. Ask yourself: at each screen size, roughly what fraction of the screen does this image take up?

```
Full width image (hero, banner)      → 100vw
Half the screen (2-col layout)       → 50vw
A third (3-col grid)                 → 33vw
A quarter (4-col grid)               → 25vw
Fixed pixel size (sidebar avatar)    → 80px
```

Then stack them smallest-screen first:

```jsx
<Image
  src={src}
  alt={alt}
  fill
  sizes="(max-width: 640px) 100vw,   // full width on mobile
         (max-width: 1024px) 50vw,   // half on tablet
         33vw"                        // third on desktop
  className="object-cover"
/>
```

The last value (no media query) is the fallback for anything larger than your final breakpoint.

Rough fractions are fine — the browser picks the nearest srcset entry anyway. You don't need exact `calc()` math.

### What actually gets downloaded at each size

Next.js pre-generates a set of widths (the srcset). The browser reads your `sizes`, works out how many pixels it needs, then picks the smallest srcset entry that covers it:

```
390px phone, 100vw → needs 390px
srcset: ...256w, 384w, 640w, 750w...
picks: 384w  ← smallest one that covers 390px

768px tablet, 50vw → needs 384px
srcset: ...256w, 384w, 640w...
picks: 384w

1400px desktop, 33vw → needs ~462px
srcset: ...384w, 640w...
picks: 640w  ← nearest above 462px

Retina phone (2× density), 390px, 100vw → needs 780px (390 × 2)
picks: 828w or nearest above
```

Without `sizes` the browser assumes `100vw` on the largest possible screen and picks `1920w` for every slot — including a card that only needed `384w`.

> **💡 Tip:** DevTools → Network → Img filter. On a phone you should see a small `w=` value matching roughly what you wrote. If you still see `w=1920` on mobile, `sizes` is missing or wrong.

### Do I need to think about this for every image?

Using the same `sizes` everywhere is already 80% of the win — much better than nothing. But it will over-download in some cases if the value doesn't match reality.

In practice you only need two decisions: is it full width, or is it in a grid?

> **📐 Sizes quick reference:**
>
> ```
> Hero / banner / full-bleed         → sizes="100vw"
> 2-col layout                       → sizes="(max-width: 640px) 100vw, 50vw"
> 3 or 4-col grid                    → sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"
> Small fixed UI (avatar, icon)      → skip sizes, use width + height instead
> ```
>
> The `33vw` default is a solid fallback for most cases. Just swap to `100vw` for heroes and you're covered.

---

## Step 3 — Add a Placeholder

This handles the gap between the card rendering and the image bytes finishing download — the UI exists, but the image slot is still empty. Two options:

| | Option A — Shimmer + fade | Option B — Blur |
|---|---|---|
| **How it looks** | Animated sweep, fades in | Blurry image comes into focus |
| **Component type** | ❌ Client Component required | ✅ Works in Server Components |
| **JS needed** | ✅ useState + onLoad | ❌ None |
| **Setup** | Manual | `FALLBACK_BLUR` constant |

If your component is already a Client Component, either works. If it's a Server Component, go straight to Option B — adding shimmer would force the whole component client-side just for a loading effect.

### Option A — Shimmer + fade

Shows an animated shimmer sweep while the image downloads, then fades the image in on load:

```jsx
"use client";
import { useState } from "react";

export default function ImageCard({ src, alt }) {
  const [loaded, setLoaded] = useState(false);

  return (
    <div className="relative w-full h-60 overflow-hidden bg-gray-100">
      {!loaded && (
        <div className="absolute inset-0 animate-pulse bg-gray-200">
          <div className="absolute inset-0 -translate-x-full animate-[shimmer_1.5s_infinite] bg-gradient-to-r from-transparent via-white/60 to-transparent" />
        </div>
      )}
      <Image
        src={src}
        alt={alt}
        fill
        sizes="(max-width: 640px) 100vw, 50vw"
        className={`object-cover transition-opacity duration-500 ${loaded ? "opacity-100" : "opacity-0"}`}
        onLoad={() => setLoaded(true)}
      />
    </div>
  );
}
```

### Option B — Blur placeholder

Good for editorial, blogs, photography — where the image itself matters.

Start by defining `FALLBACK_BLUR` once in your utils — a 1×1 grey pixel encoded as base64. It arrives in the HTML, paints instantly, and costs nothing:

```ts
// utils/constants.ts
export const FALLBACK_BLUR =
  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg=="
```

Then use it directly:

```jsx
import { FALLBACK_BLUR } from "@/utils/constants"

<Image
  src={src}
  alt={alt}
  fill
  sizes="(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 25vw"
  placeholder="blur"
  blurDataURL={FALLBACK_BLUR}
  className="object-cover"
/>
```

**Want a custom brand color instead of grey** — go to [png-pixel.com](https://png-pixel.com), pick any color, copy the base64 string. Replace the `FALLBACK_BLUR` value with it. Same behaviour, your color.

**Want the blur to actually match each image** — install Plaiceholder and generate it server-side. See [Step 5](#step-5--blur-at-scale-gallery-or-product-grid) — at that point you'd swap `FALLBACK_BLUR` for the generated `blurDataURL` per image.

---

## Step 4 — `preload` for Above-the-Fold Images

Only do this for images that are **visible on initial page load** — the first image the user sees without scrolling. Skip it for everything else.

```jsx
// ✅ First visible image — above the fold
<Image src={src} alt={alt} fill preload={true} sizes="100vw" />

// ❌ Everything else — let lazy loading do its job
<Image src={src} alt={alt} fill sizes="..." />
```

Adding `preload` to every image defeats lazy loading entirely — the browser fetches them all at once on page load regardless of whether the user ever scrolls to them.

> `priority` was the old prop — deprecated in Next.js 16. Use `preload`.

---

## Step 5 — Blur at Scale (Multiple Remote Images)

If you want real matching blur for multiple remote images, generate it server-side before passing data down to your components. The idea is the same as a single image — you just run it across an array in parallel.

### The utility

```ts
// lib/getBase64.ts
import { getPlaiceholder } from "plaiceholder"

export async function getBase64(imageUrl: string) {
  try {
    const buffer = await fetch(imageUrl).then(res => res.arrayBuffer())
    const { base64 } = await getPlaiceholder(Buffer.from(buffer))
    return base64
  } catch {
    return undefined
  }
}
```

### Single image (Server Component)

```tsx
import { getBase64 } from "@/lib/getBase64"

export default async function HeroImage({ src, alt }) {
  const blur = await getBase64(src)
  return (
    <Image src={src} alt={alt} fill placeholder="blur" blurDataURL={blur} sizes="100vw" />
  )
}
```

### Multiple images (run in parallel, not one by one)

Fetch your data, then attach a blur to each item before passing the array to your components:

```tsx
// app/your-page/page.tsx — Server Component
import { getBase64 } from "@/lib/getBase64"

export default async function Page() {
  const items = await fetchItems()  // whatever your data fetch is

  // Promise.all runs all blur fetches in parallel
  const itemsWithBlur = await Promise.all(
    items.map(async (item) => ({
      ...item,
      blurDataURL: await getBase64(item.imageUrl),
    }))
  )

  return <YourList items={itemsWithBlur} />
}
```

Your component receives `blurDataURL` as a prop and uses it directly:

```tsx
// YourComponent — receives blurDataURL as a prop
export default function YourComponent({ src, alt, blurDataURL }) {
  return (
    <Image
      src={src}
      alt={alt}
      fill
      placeholder="blur"
      blurDataURL={blurDataURL}
      sizes="(max-width: 640px) 100vw, 50vw"
      className="object-cover"
    />
  )
}
```

The flow:

```
Page (server)
    ├── fetchItems()
    └── Promise.all → getBase64() for each item (parallel)
              └── itemsWithBlur → YourList → YourComponent → blurDataURL on Image
```

---

## Where to Stop

| You have | Stop at |
|---|---|
| A quick internal tool | [Step 2 (`sizes`)](#step-2--add-sizes-most-important) |
| Images in a grid or feed | [Step 3A (shimmer + fade)](#option-a--shimmer--fade) |
| An editorial / blog | [Step 3B (blur + `FALLBACK_BLUR`)](#option-b--blur-placeholder) |
| A photography portfolio | [Step 5 (Plaiceholder per image)](#step-5--blur-at-scale-multiple-remote-images) |
| A hero + images below | [Step 4 (preload hero)](#step-4--preload-for-above-the-fold-images) + [Step 3A (shimmer for the rest)](#option-a--shimmer--fade) |