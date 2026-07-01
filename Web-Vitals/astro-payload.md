# Payload CMS — Architecture & Deployment Reference

A generic reference for integrating **Payload CMS** as a headless backend with any frontend framework (Astro, Next.js, etc.).

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Setting Up Payload CMS](#2-setting-up-payload-cms)
3. [Creating Collections](#3-creating-collections)
4. [Fetching Data from Your Frontend](#4-fetching-data-from-your-frontend)
   - [Astro](#astro)
   - [Next.js](#nextjs)
5. [Running Both Servers Locally](#5-running-both-servers-locally)
6. [Environment Variables](#6-environment-variables)
7. [Deployment](#7-deployment)
8. [Keeping Static Sites in Sync (Webhooks)](#8-keeping-static-sites-in-sync-webhooks)
9. [Enterprise-Level Architecture](#9-enterprise-level-architecture)
10. [Cost Reference](#10-cost-reference)

---

## 1. Architecture Overview

Payload runs as a **separate backend server**. Unlike Next.js (which can co-locate Payload), all other frontends (Astro, Remix, SvelteKit, etc.) must call Payload's REST or GraphQL API over HTTP.

```
┌──────────────────────────────┐        ┌──────────────────────────────┐
│  Payload CMS  (backend)      │        │  Frontend  (e.g. Astro)      │
│  Admin panel + REST API      │◄──────►│  Fetches from Payload API    │
│  Port :5000 locally          │  HTTP  │  Port :4321 locally          │
│  + PostgreSQL (e.g. Neon)    │        │  Vercel / Netlify / CF Pages │
└──────────────────────────────┘        └──────────────────────────────┘
```

**Key principle:** They are two independent projects, deployed to two separate places.

---

## 2. Setting Up Payload CMS

Scaffold a new Payload project in its own folder (separate from your frontend):

```bash
npx create-payload-app@latest
```

Prompts to answer:
- **Project name** — e.g. `my-app-cms`
- **Template** — `blank` or `blog` (blog gives you a Posts collection out of the box)
- **Database** — PostgreSQL (recommended: [Neon](https://neon.tech) for a free managed instance)

This gives you a standalone Payload server with an admin panel at `/admin`.

---

## 3. Creating Collections

Collections are the data models of your CMS. Create one in `src/collections/`:

```ts
// src/collections/Posts.ts
import { CollectionConfig } from 'payload/types'

const Posts: CollectionConfig = {
  slug: 'posts',
  admin: {
    useAsTitle: 'title',
  },
  fields: [
    {
      name: 'title',
      type: 'text',
      required: true,
    },
    {
      name: 'slug',
      type: 'text',
      required: true,
      unique: true,
    },
    {
      name: 'content',
      type: 'richText',
    },
    {
      name: 'publishedAt',
      type: 'date',
    },
  ],
}

export default Posts
```

Register it in `payload.config.ts`:

```ts
import { buildConfig } from 'payload/config'
import Posts from './collections/Posts'
import Users from './collections/Users'

export default buildConfig({
  collections: [Users, Posts],
  db: postgresAdapter({
    pool: { connectionString: process.env.DATABASE_URI },
  }),
  // ...
})
```

> **Tip:** You can create as many collections as needed — Pages, Products, Authors, Categories, etc. Each one automatically gets full CRUD REST & GraphQL endpoints.

---

## 4. Fetching Data from Your Frontend

Payload automatically exposes a REST API at `/api/{collection-slug}`.

Common endpoints:

| Action | Endpoint |
|---|---|
| List all docs | `GET /api/posts` |
| Get one by ID | `GET /api/posts/:id` |
| Filter/query | `GET /api/posts?where[slug][equals]=my-post` |
| Create | `POST /api/posts` |

### Astro

**List page** (`src/pages/index.astro`):

```astro
---
const res = await fetch(`${import.meta.env.PAYLOAD_URL}/api/posts`)
const { docs: posts } = await res.json()
---

<ul>
  {posts.map((post) => (
    <li>
      <a href={`/posts/${post.slug}`}>{post.title}</a>
    </li>
  ))}
</ul>
```

**Dynamic routes** (`src/pages/posts/[slug].astro`):

```astro
---
export async function getStaticPaths() {
  const res = await fetch(`${import.meta.env.PAYLOAD_URL}/api/posts?limit=100`)
  const { docs } = await res.json()

  return docs.map((post) => ({
    params: { slug: post.slug },
    props: { post },
  }))
}

const { post } = Astro.props
---

<article>
  <h1>{post.title}</h1>
  <div set:html={post.content_html} />
</article>
```

### Next.js

**Page with data fetching** (`app/page.tsx` — App Router):

```tsx
// Server Component — fetch runs at build time (SSG) or per request (SSR)
export default async function HomePage() {
  const res = await fetch(`${process.env.PAYLOAD_URL}/api/posts`, {
    next: { revalidate: 60 }, // ISR: revalidate every 60 seconds
  })
  const { docs: posts } = await res.json()

  return (
    <ul>
      {posts.map((post: any) => (
        <li key={post.id}>
          <a href={`/posts/${post.slug}`}>{post.title}</a>
        </li>
      ))}
    </ul>
  )
}
```

**Dynamic routes** (`app/posts/[slug]/page.tsx`):

```tsx
export async function generateStaticParams() {
  const res = await fetch(`${process.env.PAYLOAD_URL}/api/posts?limit=100`)
  const { docs } = await res.json()
  return docs.map((post: any) => ({ slug: post.slug }))
}

export default async function PostPage({ params }: { params: { slug: string } }) {
  const res = await fetch(
    `${process.env.PAYLOAD_URL}/api/posts?where[slug][equals]=${params.slug}`
  )
  const { docs } = await res.json()
  const post = docs[0]

  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content_html }} />
    </article>
  )
}
```

> **Co-location note:** If your frontend is Next.js, you *can* run Payload inside the same Next.js project using `@payloadcms/next`. This removes the need for two separate servers, but the separate-server approach described here is framework-agnostic and always works.

---

## 5. Running Both Servers Locally

You need two terminal sessions:

```bash
# Terminal 1 — Payload CMS backend
cd my-app-cms
npm run dev        # → http://localhost:5000
                   # Admin panel: http://localhost:5000/admin
```

```bash
# Terminal 2 — Frontend
cd my-app-frontend
npm run dev        # → http://localhost:4321  (Astro)
                   #   http://localhost:3000  (Next.js)
```

---

## 6. Environment Variables

### Payload CMS (`.env`)

```env
DATABASE_URI=postgresql://user:password@host/dbname
PAYLOAD_SECRET=your-random-secret-string-here
```

### Frontend (`.env`)

```env
# Local
PAYLOAD_URL=http://localhost:5000

# Production (update after deploying Payload)
PAYLOAD_URL=https://your-cms.up.railway.app
```

Usage in Astro: `import.meta.env.PAYLOAD_URL`
Usage in Next.js: `process.env.PAYLOAD_URL`

> **Security:** Never expose your `PAYLOAD_SECRET` or database credentials to the frontend. Only `PAYLOAD_URL` is needed client-side.

---

## 7. Deployment

### Deploy Payload CMS (backend)

Payload needs a Node.js server environment. Recommended platforms:

| Platform | Notes | Cost |
|---|---|---|
| [Railway](https://railway.app) | Easiest, auto-detects Node.js | Free / ~$5/mo |
| [Render](https://render.com) | Similar to Railway, free tier spins down | Free / $7/mo |
| [Vercel](https://vercel.com) | Works with `@payloadcms/next` (co-located only) | Free |
| [Fly.io](https://fly.io) | More control, Docker-based | Free tier |

Steps for **Railway** (recommended for standalone Payload):

1. Push your CMS repo to GitHub
2. Create a new project on Railway → **Deploy from GitHub repo**
3. Add environment variables (`DATABASE_URI`, `PAYLOAD_SECRET`)
4. Railway auto-detects `npm run start` and deploys

### Deploy the Frontend

Any static or SSR-capable host works:

| Platform | Best for | Cost |
|---|---|---|
| [Vercel](https://vercel.com) | Next.js (first-class support), Astro SSR | Free |
| [Netlify](https://netlify.com) | Astro, SvelteKit | Free |
| [Cloudflare Pages](https://pages.cloudflare.com) | Edge rendering, Astro | Free |

Steps (all platforms follow the same pattern):

1. Push your frontend repo to GitHub
2. Connect the repo on your chosen platform
3. Set build command (`npm run build`) and output directory (`dist` for Astro, `.next` for Next.js)
4. Add environment variable: `PAYLOAD_URL=https://your-deployed-cms-url.com`

---

## 8. Keeping Static Sites in Sync (Webhooks)

If your frontend uses **SSG (Static Site Generation)**, it won't reflect new CMS content until it rebuilds. Set up a webhook to automate this:

### Step 1 — Get a Deploy Hook URL

**Vercel:** Project Settings → Git → Deploy Hooks → Create Hook → copy the URL

**Netlify:** Site Settings → Build & Deploy → Build Hooks → Add Hook → copy the URL

### Step 2 — Add a Webhook in Payload

In `payload.config.ts`:

```ts
export default buildConfig({
  hooks: {
    // Or configure per-collection in the collection config
  },
  collections: [
    {
      slug: 'posts',
      hooks: {
        afterChange: [
          async () => {
            await fetch(process.env.FRONTEND_DEPLOY_HOOK_URL!, { method: 'POST' })
          },
        ],
      },
      fields: [ /* ... */ ],
    },
  ],
})
```

Now every time an editor publishes or updates content in Payload, your frontend automatically rebuilds and deploys.

> **Alternative:** For sites that need real-time updates without rebuilds, switch to **SSR** or use Next.js **ISR** (Incremental Static Regeneration) with `next: { revalidate: N }` on your fetch calls.

---

## 9. Enterprise-Level Architecture

For larger apps with multiple surfaces (dashboard, marketing site, API), the pattern expands:

```
apps/
├── web/          ← Next.js marketing site or dashboard  →  deploy to Vercel
├── cms/          ← Payload CMS (admin + REST API)       →  deploy to Vercel or Railway
└── api/          ← Hono / Express API layer (optional)  →  deploy to Railway or Fly.io

packages/
├── ui/           ← Shared component library
├── types/        ← Shared TypeScript types (Payload-generated)
└── config/       ← Shared ESLint, TypeScript config
```

This is typically managed as a **monorepo** using Turborepo or pnpm workspaces.

**Payload can auto-generate TypeScript types** for your collections:

```bash
npx payload generate:types
```

Share the generated `payload-types.ts` across your apps via a shared `packages/types` package to keep everything type-safe end-to-end.

---

## 10. Cost Reference

| Service | What it's for | Cost |
|---|---|---|
| Railway / Render | Payload CMS server | Free tier / $5–7/mo |
| [Neon](https://neon.tech) | PostgreSQL database | Free (0.5 GB) |
| Vercel / Netlify | Frontend hosting | Free |
| [Cloudflare R2](https://developers.cloudflare.com/r2/) / S3 | Media uploads & file storage | Free tier (10 GB) |

> **Total for a typical small project: $0/mo** on free tiers, scaling to ~$10–15/mo as traffic grows.

---

## Quick Reference: Full Architecture Diagram

```
┌───────────────────────────────────────────────────────────────────────┐
│                        PRODUCTION DEPLOYMENT                          │
│                                                                       │
│  ┌─────────────────────┐     REST/GraphQL     ┌──────────────────┐   │
│  │   Payload CMS        │◄──────────────────►  │   Frontend App   │   │
│  │   Railway / Render   │                      │   Vercel /       │   │
│  │   + Neon (Postgres)  │                      │   Netlify / CF   │   │
│  └──────────┬──────────┘                      └──────────────────┘   │
│             │ Webhook on publish                        ▲             │
│             └──────────────────── Trigger rebuild ──────┘             │
│                                                                       │
│  ┌─────────────────────┐                                              │
│  │   Media Storage      │  ← Cloudflare R2 or AWS S3                  │
│  └─────────────────────┘                                              │
└───────────────────────────────────────────────────────────────────────┘
```