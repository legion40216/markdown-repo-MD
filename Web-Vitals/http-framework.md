# HTTP Server Frameworks
web framewroks / backend frameworks/ server frameworks / http framework

## REST is not the only API method that HTTP Server Frameworks work on
HTTP server framework is not REST framework, REST is one of the many api methods that these server can work on. 

Api methods work on the transport layer.

```
Express
│
├── REST      →  app.get("/users", ...)
├── GraphQL   →  app.post("/graphql", ...)
├── RPC       →  app.post("/rpc/user.getAll", ...)
└── Custom    →  whatever you want
```

Express became associated with REST only because that's how most people historically used it — not because of any technical limitation.

A cleaner mental model separates the two layers:

```
Transport Layer          API Layer
────────────────         ────────────────
Express                  REST
Hono            +        GraphQL
Fastify                  RPC
Next.js
```

---

## HTTP frameworks

### Hono 

```ts
const app = new Hono();

app.get("/users", ...)
app.post("/users", ...)
app.put("/users/:id", ...)
```

It's an HTTP framework, similar to Express or Fastify, but modern.

Its primary concern is:

* routing
* middleware
* request handling
* HTTP responses

---

### Next.js

Next.js is an HTTP server

When you run:

next dev

Next.js starts an HTTP server that handles requests.

Normally:

Browser
   │
   ▼
Next.js Server
   │
   ├── Pages
   ├── App Router
   └── API Routes

### Nest.js 
This is standard setup. Two independent projects, each with its own package.json, each running its own dev server on its own port.

```
my-project/
├── backend/          (NestJS — runs on e.g. localhost:3001)
│   ├── src/
│   ├── package.json
│   └── nest-cli.json
└── frontend/          (Next.js — runs on e.g. localhost:3000)
    ├── app/
    ├── package.json
    └── next.config.js
```

You'd start each separately:
bash# terminal 1
cd backend && npm run start:dev   # Nest on :3001

#### part 2
cd frontend && npm run dev        # Next.js on :3000
Next.js then just calls Nest like any external API, usually from server components, route handlers, or client-side fetches:
tsconst res = await fetch('http://localhost:3001/api/users')
In production you'd point that at your deployed Nest URL via an env var (NEST_API_URL or similar) instead of hardcoding localhost.
A few practical notes on this setup:

CORS — since they're on different origins, you'll need to enable CORS in Nest (app.enableCors() in main.ts) or proxy through Next's rewrites() config in next.config.js to make it look same-origin from the browser.
Monorepo tooling (optional) — if you want them in one repo but still separate apps, tools like Turborepo or Nx let you keep two folders like above but manage shared scripts, shared TypeScript types, and concurrent dev servers (turbo dev starts both at once) without merging the actual runtimes.
Deployment — they deploy independently too: Nest typically goes to a regular Node host (Railway, Render, a VPS, Fly.io, ECS, etc.) since it needs a persistent Node process, while Next.js can go to Vercel or wherever. They don't have to live on the same platform.
Shared types — a common pain point is keeping request/response shapes in sync between the two. People either hand-write shared TS types in a shared package, or generate an OpenAPI spec from Nest (it has built-in Swagger support) and codegen a typed client for the Next.js side.

Bonus Tip

Some developers actually combine both:

Use Hono for REST endpoints or authentication middleware
Use tRPC for internal type-safe communication between front and back

That’s a powerful combo for full-stack TypeScript apps.
---

## Elysia 

##Elysia + Eden Treaty Standlone server (Bun)
```ts
// Elysia server (standalone Bun server)
const app = new Elysia()
  .get('/users', () => [])

// Eden client (fully typed)
import { treaty } from '@elysiajs/eden'
const client = treaty<typeof app>('localhost:3000')
const { data } = await client.users.get() // fully typed
```

A simple snppit to start Elysia

##Elysia + Eden Treaty integration with Fullstack frameworks(Nextjs)
Elysia integration works through a clever combination of Next.js's file-system routing and ElysiaJS's design. Here's how it all fits together:

**The catch-all route is the key**

The file `app/api/[...slug]/route.ts` is a Next.js **catch-all API route**. The `[...slug]` syntax means it intercepts *every* request to `/api/*` — so `/api/users`, `/api/posts/123`, etc. all flow into that single file.

**ElysiaJS handles it from there**

Inside that file, you create an Elysia app instance and export its HTTP methods:

```ts
// app/api/[...slug]/route.ts
import { Elysia } from 'elysia'

const app = new Elysia({ prefix: '/api' })
  .get('/hello', () => 'Hello NextJS')
  .post('/users', ...)

export const GET = app.handle
export const POST = app.handle
export const PUT = app.handle
export const DELETE = app.handle
```

Next.js expects named exports (`GET`, `POST`, etc.) from route files — and `app.handle` is just a function that accepts a standard `Request` and returns a `Response`. Since both sides speak the **Web Fetch API standard**, they're fully compatible.

Because the same Elysia instance defines both the routes *and* exports its type, Eden can infer the exact shape of every endpoint:

```ts
// lib/eden.ts
import { treaty } from '@elysiajs/eden'
import type { App } from '@/app/api/[...slug]/route'

export const api = treaty<App>('localhost:3000')
```

Now when you call `api.hello.get()` on the frontend, TypeScript *already knows* the return type is `string` — no codegen, no OpenAPI spec needed.

**Why it works so cleanly**

The whole thing is possible because of three converging factors:

1. **Next.js Route Handlers** use the standard `Request`/`Response` API, not Node's `req`/`res`
2. **ElysiaJS** was built around that same standard (originally for Bun, which is also standards-based)
3. **TypeScript's type imports** let Eden reference the server's types at zero runtime cost — the type information is stripped at build time

So essentially, Next.js acts as the *host*, the catch-all route acts as a *gateway*, and Elysia runs as a fully self-contained router *inside* that gateway.

---Key notes
Elysia with Next.js — this is where it breaks down:

* Elysia is Bun-first, designed to be a standalone server
* It doesn't have an official Next.js/Vercel adapter
* Next.js runs on Node.js (or Edge), not Bun
* So you can't mount Elysia inside Next.js route handlers the way you can with "Hono"
Elysia does have its own "RPC" called Eden Treaty which is similar to "tRPC":
ts
---

**What actually works**

Elysia *can* run inside Next.js route handlers in a limited sense — because of the Web Fetch API compatibility you noted for Hono. The `app.handle` pattern isn't fictional:

```ts
export const GET = app.handle
```

This *does* work on Node.js with recent Elysia versions, because Elysia added a compatibility layer for non-Bun runtimes. But it comes with caveats.

**Where it actually breaks down**

- **Performance**: Elysia's speed benchmarks are Bun-specific. On Node.js you lose most of that advantage
- **Some Elysia plugins** assume Bun APIs (`Bun.file`, `Bun.serve`, etc.) and will break on Node
- **Eden Treaty's type inference** still works because it's purely TypeScript — but the *runtime* is now Node, not Bun, so you're running Elysia in a mode it wasn't designed for
- **Vercel deployment** adds another layer — Edge runtime won't run Elysia at all, and even the Node runtime has constraints

It probably *does run* — Elysia doesn't hard-crash on Node.js. But it's a demo environment, not a production recommendation. If you actually want Elysia + Eden Treaty the way it's designed to work, you'd run Elysia as a **standalone Bun server** alongside Next.js, not inside it.

Yes! Here's why:

---

## Category 2 — Custom Server (wraps Next.js)
Elysia can boot Next.js inside it, acting as the main server:
```ts
// server.ts
import { Elysia } from 'elysia'
import next from 'next'

const app = next({ dev: process.env.NODE_ENV !== 'production' })
const handle = app.getRequestHandler()

await app.prepare()

new Elysia()
  // Your own Elysia API routes
  .get('/api/hello', () => ({ message: 'Hello from Elysia!' }))

  // Everything else goes to Next.js
  .all('*', ({ request }) => handle(request))

  .listen(3000)
```
Same concept as Express/Fastify custom server — Elysia is the boss, Next.js is the passenger.

---

## Category 3 — Separate Backend Service
Elysia runs on its own port completely independently, and Next.js just calls it via `fetch()`:
```ts
// Elysia backend — runs on port 4000
new Elysia()
  .get('/users', () => [{ id: 1, name: 'John' }])
  .listen(4000)
```
```ts
// Next.js frontend — calls Elysia
const res = await fetch('http://localhost:4000/users')
const users = await res.json()
```
Two completely separate processes talking to each other.

---

## Summary

| Category | How Elysia fits |
|---|---|
| **Cat 2 - Custom Server** | Elysia wraps and boots Next.js |
| **Cat 3 - Separate Service** | Elysia runs standalone, Next.js fetches from it |

---

The same is actually true for **Express and Fastify** too — they all fit in both Category 2 and 3. The categories describe *how you use them*, not a limitation of the framework itself.

Good question. If Elysia is a standalone Bun server, your deployment options are:

**Platforms that support Bun natively**

- **Railway** — easiest, just point it at your repo and it detects Bun automatically
- **Render** — supports Bun as a runtime, straightforward setup
- **Fly.io** — containerized, you write a Dockerfile with `FROM oven/bun` and deploy
- **VPS (DigitalOcean, Hetzner, etc.)** — install Bun on the server, run it yourself with a process manager like PM2 or just `bun run`

**The architecture then looks like this**

```
┌─────────────────┐        ┌──────────────────┐
│   Next.js app   │  HTTP  │  Elysia (Bun)    │
│   (Vercel)      │ ──────▶│  (Railway/Render)│
│                 │        │                  │
│  Eden Treaty    │        │  your API        │
└─────────────────┘        └──────────────────┘
```

Your Next.js frontend stays on Vercel, and Eden Treaty points to the external Elysia URL instead of `localhost`:

```ts
// lib/eden.ts
const client = treaty<typeof app>(process.env.NEXT_PUBLIC_API_URL)
```

**The tradeoff**

You now have two deployments to manage instead of one. That's the real cost. This is why most people reaching for a typed RPC layer inside a Next.js monorepo just pick **tRPC or Hono** — same type safety, no separate server needed.

Elysia + Eden makes more sense if you *want* a separate backend (e.g. a microservice, or a non-Next.js frontend later).

## Eden (by ElysiaJS)

**Eden** is the official end-to-end typesafe RPC client for **[ElysiaJS](https://elysiajs.com/)** — a Bun-first web framework. Think of it as tRPC but for Elysia.

---

### How it works

You define your API with Elysia, and Eden automatically infers all types — no schema duplication needed.

```ts
// server.ts (Elysia app)
import { Elysia, t } from 'elysia';

const app = new Elysia()
  .get('/hello', () => 'Hello World')
  .post('/user', ({ body }) => body, {
    body: t.Object({
      name: t.String(),
      age: t.Number(),
    }),
  })
  .listen(3001);

export type App = typeof app; // 👈 export the type
```

```ts
// client.ts (anywhere in Next.js)
import { treaty } from '@elysiajs/eden';
import type { App } from '../server';

const client = treaty<App>('localhost:3001');

// Fully typed! ✅
const { data } = await client.hello.get();
const { data: user } = await client.user.post({ name: 'Ali', age: 25 });
```

---

### Eden Treaty vs Eden Fetch

| | **Eden Treaty** | **Eden Fetch** |
|---|---|---|
| Style | `client.user.get()` (object chaining) | `edenFetch('/user', {})` (fetch-like) |
| DX | ⭐ Better | Simpler |
| Recommended | ✅ Yes | For simple cases |

---

### Separate Server with Next.js

Since Elysia runs on **Bun**, the typical pattern is a **separate microservice** architecture:

```
Next.js (Node/Vercel)  ←→  Elysia server (Bun)
     [frontend]                  [API/backend]
```

```ts
// next.js app — just import the type, not the server
import { treaty } from '@elysiajs/eden';
import type { App } from '../../elysia-server/src/index'; // type-only import

const api = treaty<App>(process.env.API_URL!);

// In a Server Component or API route:
export async function GET() {
  const { data, error } = await api.users.get();
  return Response.json(data);
}
```

> **Key point:** You only import the **type** from Elysia — no runtime Bun code runs in Next.js.


---

## Elysia + Eden Treaty on Separate Servers

```ts
// Backend — Elysia on port 4000 (separate server)
const app = new Elysia()
  .get('/users', () => [{ id: 1, name: 'John' }])
  .listen(4000)

export type App = typeof app // 👈 Export the type
```

```ts
// Frontend — Next.js on port 3000
import { treaty } from '@elysiajs/eden'
import type { App } from '../backend' // 👈 Import the type

const client = treaty<App>('localhost:4000')

const { data } = await client.users.get()
//                                   ^ TypeScript knows this returns
//                                     { id: number, name: string }[]
```

They are on **completely separate servers** but still fully type-safe. The only thing being shared is the **TypeScript type**, not the actual code.

---

### 4. Elysia with Eden — The Bun Native Option

Elysia is to Bun what Hono is to the edge. It has its own RPC system called Eden:

```javascript
// Elysia server
const app = new Elysia()
  .get('/users/:id', ({ params }) => ({
    name: 'Ahmed',
    id: params.id
  }))

// Eden client — fully typed
const { data } = await eden.users({ id: '42' }).get()
```


## Elysia + Eden

**Eden** is Elysia's official end-to-end type-safe client.

Like Hono RPC, Eden infers the types of your Elysia routes and provides a fully typed client without code generation.

```ts
client.users.get()
client.users.post({ name: "John" })
```

Although the API feels like RPC, it still communicates using standard HTTP. The only thing being shared is the TypeScript types.

**Summary**

✅ Full backend framework  
✅ Bun optimized  
✅ End-to-end type safety  
⚠️ Manual TanStack Query integration

---

# Why Doesn't Every Framework Do This?

The difference comes down to **how the framework represents routes**.

Traditional frameworks like Express were designed long before end-to-end type inference became a goal.

```ts
app.get('/users', (req, res) => {
  res.json({ id: 1, name: 'John' })
})

export type App = typeof app
```

TypeScript can infer types *inside* the callback, but the Express application itself doesn't preserve enough information for another tool to automatically reconstruct the API.

As a result:

```ts
export type App = typeof app
```

doesn't contain meaningful route metadata that a typed client can consume.

---

# Elysia Was Designed for Type Inference

Elysia's routing system was designed so that route definitions preserve enough compile-time information for Eden to infer both request and response types.

```ts
const app = new Elysia()
  .get('/users', () => [
    { id: 1, name: 'John' }
  ])

export type App = typeof app
```

Because the route definition itself carries rich type information, Eden can automatically generate a fully typed client.

No code generation.

No OpenAPI specification.

No duplicated interfaces.

---

# Bottom Line

> Any framework can share TypeScript types manually, but frameworks like **Elysia** and **Hono** were designed so that route definitions preserve enough compile-time type information for their official clients (Eden and Hono RPC) to infer the API automatically.

That's the real difference.

It's not magic, nor a different transport protocol—it's simply a framework design that works exceptionally well with TypeScript's type system.

---

# Limitations of Eden

Like Hono RPC, Eden intentionally focuses on providing a type-safe HTTP client rather than a complete full-stack data-fetching solution.

## 1. Manual Data Fetching

Eden provides fully typed HTTP methods:

```ts
await client.users.get()
await client.users.post({
  name: "John"
})
```

How you fetch, cache, retry, or synchronize data is left to whichever data-fetching library you choose.

For example, with TanStack Query you'll typically write the query yourself:

```ts
useQuery({
  queryKey: ['users'],
  queryFn: async () => {
    const { data } = await client.users.get()
    return data
  }
})
```

Eden does not generate React Query hooks automatically.

---

## 2. Communication Still Uses HTTP

Even though the client is fully typed, requests are still normal HTTP requests.

```
Frontend
     │
 HTTP Request
     │
     ▼
 Elysia
```

Standard HTTP concerns still apply:

- Headers
- Cookies
- Authentication
- Serialization
- Network latency

Type safety does not eliminate HTTP—it simply makes it safer to use.

---

## 3. Authentication Works Like Any Other HTTP API

Authentication follows normal web API patterns.

Requests may require:

- Cookies
- Authorization headers
- Bearer tokens
- Sessions

Eden does not automatically forward authentication information between separate HTTP requests.

---

## 4. Bun-First Design

Elysia was originally designed around the Bun runtime.

Although it now supports Node.js, some features and plugins remain Bun-oriented, and most of its performance advantages are realized when running on Bun.

For that reason, the most common production architecture is:

```
Next.js (Node/Vercel)
          │
      HTTP Requests
          │
          ▼
Elysia + Eden (Bun)
```

rather than embedding Elysia inside another framework.

---

# Summary

| Feature | Elysia | Eden |
|----------|--------|------|
| Backend framework | ✅ | ❌ |
| Routing | ✅ | ❌ |
| Middleware | ✅ | ❌ |
| Validation | ✅ | ❌ |
| Runtime agnostic | ⚠️ Primarily Bun | ⚠️ Depends on Elysia |
| Type-safe client | ❌ | ✅ |
| Automatic type inference | ❌ | ✅ |
| Uses normal HTTP | ✅ | ✅ |
| Can run as a dedicated backend | ✅ | ✅ |
| Can be integrated into full-stack frameworks | ⚠️ Possible, but not the primary design | ✅ |
````

# Hono 
Hono

You manually define routes like a traditional HTTP API:

import { Hono } from 'hono';

const app = new Hono();

app.get('/hello', (c) => c.text('Hello Hono!'));
app.post('/users', async (c) => {
  const body = await c.req.json();
  return c.json({ user: body });
});

export default app;

✅ Best for:

API gateways
Edge/serverless environments (Cloudflare, Vercel Edge, Bun, Deno, etc.)
Small REST APIs with minimal overhead

##Hono RPC

Hono RPC (what it actually is)

Hono includes an optional RPC-style client/server helper that lets you:

define routes in Hono
generate a typed client automatically
call endpoints like functions (instead of manually writing fetch)

Example idea:

// server
app.get('/user/:id', (c) => {
  return c.json({ id: c.req.param('id') });
});

Then on the client you can do something like:

client.user[':id'].$get()

or with typed inference depending on setup.

👉 This feels “RPC-like”, but under the hood it’s still:

HTTP requests with type inference

## What Is Hono?

**Hono** is a lightweight, runtime-agnostic backend web framework for JavaScript and TypeScript.

Its primary goal is to let you write your backend once and deploy it to many different JavaScript runtimes without changing your application code.

Supported runtimes include:

- Node.js
- Bun
- Deno
- Cloudflare Workers
- Vercel Edge Functions
- AWS Lambda
- Fastly Compute
- and many others

Instead of learning the APIs for every platform, you write a Hono application and Hono adapts it to the environment.

---

## The Problem Hono Solves

Every JavaScript runtime exposes requests and responses differently.

### Node.js

```ts
import http from 'node:http'

http.createServer((req, res) => {
  console.log(req.url)
})
```

### Cloudflare Workers

```ts
export default {
  fetch(request: Request) {
    return new Response("Hello")
  }
}
```

### AWS Lambda

```ts
export const handler = async (event, context) => {
  // different request object again
}
```

Although they all receive HTTP requests, each runtime exposes different APIs.

Hono provides one consistent programming model regardless of where your application runs.

```
Your Code
     │
     ▼
   Hono
     │
 ┌───┼───────────────┐
 │   │       │       │
Node Edge  Workers Lambda
```

---

# What Does Hono Actually Do?

Hono is responsible for common backend tasks such as:

- Routing
- Middleware
- Reading requests
- Returning responses
- Validation
- Cookies
- Context handling

Example:

```ts
import { Hono } from "hono"

const app = new Hono()

app.get("/users", (c) => {
  return c.json([
    { id: 1, name: "John" }
  ])
})
```

This looks very similar to Express or Fastify.

---

# Hono Is Flexible

Unlike some frameworks, Hono doesn't require owning the entire server.

It can either:

- be your entire backend
- or be mounted inside another framework

That flexibility gives you two common deployment styles.

---

# Option 1 — Hono as a Dedicated Backend Server

In this architecture, Hono is its own application.

```
Frontend
     │
HTTP Requests
     │
     ▼
Hono Server
     │
Database
```

Example:

```
frontend.example.com
api.example.com
```

or

```
Next.js
      │
      ▼
api.mycompany.com (Hono)
```

The frontend communicates with Hono over HTTP just like any other API.

### Advantages

- Completely independent backend
- Can serve multiple frontends
- Mobile apps can reuse the API
- Easier to scale separately
- Can deploy independently

### Good for

- Public APIs
- Multiple clients
- Mobile + Web
- Microservices
- Team separation

---

# Option 2 — Hono Inside a Meta Framework

Instead of deploying another server, Hono can live inside frameworks such as:

- Next.js
- Astro
- Remix
- Other frameworks supporting request handlers

For Next.js, Hono is commonly mounted under `/api`.

```
Next.js
│
├── Pages
├── React Components
├── Server Components
└── Hono App
      │
      └── /api/*
```

Example:

```text
/api/users
/api/posts
/api/comments
```

are all handled by Hono.

---

## Example

```ts
import { Hono } from "hono"
import { handle } from "hono/vercel"

const app = new Hono().basePath("/api")

app.get("/users", (c) => {
  return c.json([])
})

export const GET = handle(app)
```

In Next.js, this is typically placed inside:

```
app/api/[[...route]]/route.ts
```

The catch-all route forwards every `/api/*` request to Hono.

---

## Why Use Hono Inside a Meta Framework?

Without Hono:

```
app/
└── api/
    ├── users/
    │     route.ts
    ├── posts/
    │     route.ts
    ├── auth/
    │     route.ts
    └── comments/
          route.ts
```

Every endpoint lives in its own file.

As applications grow, this can become difficult to organize.

With Hono:

```
app/
└── api/
    └── [[...route]]
          route.ts
```

Inside that file:

```ts
app.get("/users")
app.get("/posts")
app.post("/posts")
app.delete("/posts/:id")
```

Hono manages all routing internally.

---

# Hono RPC

## What Is Hono RPC?

Hono RPC is **not another framework**.

It is a feature built on top of Hono that allows the frontend to infer API types directly from the backend.

Instead of manually writing API clients, Hono can generate a fully typed client.

Backend:

```ts
const app = new Hono()

app.get("/users", (c) => {
  return c.json([{ id: 1 }])
})

export type AppType = typeof app
```

Frontend:

```ts
import { hc } from "hono/client"

const client = hc<AppType>("http://localhost:3000")

const res = await client.users.$get()
```

The client automatically knows:

- available routes
- parameters
- request body
- response types

without generating code.

---

# Hono vs Hono RPC

These are related but different concepts.

```
Hono
│
├── Routing
├── Middleware
├── Validation
├── Request handling
└── Response handling
```

```
Hono RPC
│
├── Everything Hono provides
└── Type-safe client generation
```

Think of Hono RPC as an optional feature layered on top of Hono.

---

# Where Does Hono RPC Work?

Hono RPC works whether Hono is:

- running as a dedicated backend
- mounted inside a meta framework

Example:

```
Frontend
      │
      ▼
Typed Hono Client
      │
      ▼
Hono Application
```

The deployment style does not change how Hono RPC works.

---

# Limitations of Hono RPC

Hono RPC provides excellent type safety, but it intentionally stays fairly lightweight.

It focuses on generating a typed HTTP client rather than providing a complete data-fetching ecosystem.

Some areas where developers often write additional code include:

---

## 1. Manual Data Fetching

Hono RPC provides typed HTTP methods:

```ts
await client.users.$get()
await client.posts.$post()
```

How you fetch, cache, retry, or synchronize data is entirely up to you.

For example, if you're using a library like TanStack Query, you'll typically write the query configuration yourself.

```ts
useQuery({
  queryKey: ["users"],
  queryFn: async () => {
    const res = await client.users.$get()
    return res.json()
  }
})
```

Hono RPC intentionally doesn't generate these query hooks.

---

## 2. HTTP Is Always Part of the Communication

Hono RPC communicates using normal HTTP requests.

```
Frontend
      │
HTTP Request
      │
      ▼
Hono
```

Even though the client is fully typed, the request still travels through HTTP.

This means standard HTTP concerns still apply, including:

- headers
- cookies
- authentication
- serialization
- network latency

Hono RPC does not bypass HTTP simply because both sides use TypeScript.

---

## 3. Authentication Context Must Be Managed Normally

Since communication happens over HTTP, authentication works like any other web API.

Requests may need:

- cookies
- authorization headers
- bearer tokens
- sessions

Hono RPC does not automatically infer or forward authentication information between different HTTP requests.

---

## 4. Focused Scope

Hono intentionally stays focused on being:

- a backend framework
- a routing system
- a typed HTTP API

It does **not** attempt to provide an all-in-one full-stack solution.

Instead, developers are free to combine it with whatever tools best fit their application.

---

# Choosing Between the Two Deployment Styles

## Dedicated Hono Server

```
Frontend
     │
HTTP
     │
     ▼
Hono
     │
Database
```

Choose this when:

- multiple clients consume the API
- mobile apps share the backend
- services are deployed independently
- teams own separate frontend and backend projects

---

## Hono Inside a Meta Framework

```
Meta Framework
│
├── UI
├── Pages
├── Components
└── Hono
      │
      └── /api/*
```

Choose this when:

- building a single web application
- you want cleaner API routing
- you don't want another deployment
- the backend exists primarily to support the current application

---

# Summary

| Feature | Hono | Hono RPC |
|----------|-------|----------|
| Backend framework | ✅ | ✅ |
| Routing | ✅ | ✅ |
| Middleware | ✅ | ✅ |
| Validation | ✅ | ✅ |
| Runtime agnostic | ✅ | ✅ |
| Type-safe client | ❌ | ✅ |
| Typed API calls | ❌ | ✅ |
| Uses normal HTTP | ✅ | ✅ |
| Can run inside meta frameworks | ✅ | ✅ |
| Can run as a dedicated backend | ✅ | ✅ |

A hybrid architecture means using Hono as the web framework and tRPC for part of your API, instead of choosing only one.

For example:

                Frontend
                    │
        ┌───────────┴───────────┐
        │                       │
   tRPC Client              fetch()/Axios
        │                       │
        ▼                       ▼
   tRPC Endpoint          REST Endpoint
        │                       │
        └───────────┬───────────┘
                    ▼
                 Hono App
                    │
          Database / Services

Here, Hono is the server, and it can expose different kinds of endpoints:

tRPC endpoints for your own TypeScript frontend
REST endpoints for everyone else
Why would you do this?

Imagine you're building an e-commerce application.

Your Next.js frontend is written in TypeScript, so you want the convenience of tRPC:

await trpc.cart.addItem.mutate(...)

But you also need:

a payment provider webhook
a mobile app written in Flutter
third-party developers consuming your API

Those cannot use the tRPC client.

So you expose normal REST endpoints too:

POST /api/webhooks/stripe
GET  /api/products
POST /api/orders

Now you get the best of both worlds:

tRPC for your own frontend (excellent developer experience and type safety)
REST for external consumers and integrations
Hono handles all the HTTP routing and middleware.
Does Hono require tRPC?

No.

Hono's own RPC client is good enough for many projects. A lot of developers simply use:

Frontend
   │
Hono RPC Client
   │
Hono Server

without any tRPC at all.

So when would you pick each?
Just Hono: You want a fast backend, REST APIs, or Hono's built-in typed RPC is enough.
Hono + tRPC: You love tRPC's procedure-based API for your TypeScript frontend but still want Hono as your server framework and may also expose REST endpoints.
Just tRPC: Typically when you're in a full-stack framework (like Next.js) and don't need a separate backend framework beyond the adapter.

The important point is that Hono and tRPC are not direct competitors. Hono is a web framework, while tRPC is an API layer. You can use either independently or together, depending on the architecture you want.