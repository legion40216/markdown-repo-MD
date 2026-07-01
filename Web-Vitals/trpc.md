ou define procedures instead of routes — the frontend automatically infers types from backend logic:

// server/trpc/router.ts
import { initTRPC } from '@trpc/server';

const t = initTRPC.create();
export const appRouter = t.router({
  getUser: t.procedure.input(z.string()).query(({ input }) => {
    return { id: input, name: 'John Doe' };
  }),
});

// frontend
const user = await trpc.getUser.query('123'); // Type-safe on both ends

✅ Best for:

Full-stack TypeScript (Next.js, SvelteKit, etc.)
Sharing types between client and server
Avoiding REST/GraphQL boilerplate

# tRPC
type-safe client-server communication api with RPC Api methodolgy

tRPC = type-safe API layer (not a server framework)
It does not create a server by itself
It defines procedures (functions) that are exposed over HTTP/WebSocket
You still need a server runtime (Express, Next.js, Hono, Fastify, etc.)

👉 It’s more like:

“a way to call backend functions from frontend with full TypeScript safety”

🧩 How tRPC actually works

tRPC sits on top of a backend:

Frontend
   ↓
tRPC Client
   ↓
HTTP/WebSocket
   ↓
tRPC Adapter (Express / Next.js / Hono)
   ↓
Your procedures (functions)
   ↓
Database / logic

So tRPC is basically:

“typed function calls over HTTP”

```
Backend defines router
      ↓
Type exported to shared package
      ↓
Next.js imports ONLY the type
      ↓
tRPC client uses that type
      ↓
Every component gets:
  ✅ RPC style calls
  ✅ Full type safety
  ✅ TanStack Query (loading, caching, mutations)
  ✅ Zero manual type maintenance
```
> The magic is that **zero backend code** ships to the frontend — only the TypeScript type travels across, giving you full safety with no coupling.

Normally when you have a React frontend and an Express backend, you write API endpoints on the server, then manually write matching fetch calls on the client, and you have to keep them in sync yourself. If you rename a field on the server, the client silently breaks at runtime. tRPC eliminates that by sharing types between both sides.

## What is tRPC?

tRPC is a **TypeScript RPC library** that adds a **type-safe API layer** on top of an existing HTTP server. It allows your frontend to call backend procedures with **end-to-end TypeScript type safety**, eliminating the need to manually define REST endpoints or API schemas.

Unlike REST or GraphQL, the frontend automatically infers the types of backend procedures, giving you autocomplete and compile-time type checking across your application.

tRPC — framework-agnostic procedures + TanStack Query

tRPC lifts the same RPC idea but makes it a **framework-agnostic adapter**. You define a router with procedures (queries, mutations, subscriptions), and tRPC plugs into whatever server you're already using — Next.js, Express, Fastify, Hono, Nuxt, anything. Switch the server, the client code doesn't change.

The bigger differentiator is what you get on the client. It's not just a type-safe `fetch` wrapper. Every procedure becomes a **real TanStack Query hook**:

```ts
// server
const appRouter = router({
  post: router({
    list:   publicProcedure.query(() => db.post.findMany()),
    create: publicProcedure
              .input(z.object({ title: z.string() }))
              .mutation(({ input }) => db.post.create({ data: input }))
  })
})

// client — these are TanStack Query hooks, fully typed
const { data, isLoading } = trpc.post.list.useQuery()
const create = trpc.post.create.useMutation()
```

`trpc.post.list.useQuery()` gives you caching, background refetching, loading and error states, stale-while-revalidate, optimistic mutations — everything TanStack Query provides, but the types come directly from the server router through TypeScript inference,

---

# Core Concepts

## Procedure

A **procedure** is simply a server-side function that is exposed as part of your API.

Instead of creating REST routes, you define procedures.

```ts
const getUser = t.procedure.query(() => {
  return {
    id: 1,
    name: "Ali",
  };
});
```

---

## Router

A **router** groups related procedures together.

It acts as the entry point to your API.

```ts
export const appRouter = t.router({
  getUser,
  createUser,
  deleteUser,
});
```

Every procedure that the frontend can call is registered inside a router.

---

## Client

The **tRPC client** lets your frontend call backend procedures as if they were local functions.

```ts
const user = await trpc.getUser.query();
```

Notice there is:

* no URL
* no HTTP method
* no `fetch()`
* no manually written request types

The client automatically inherits the types of the server procedures.

---

## End-to-End Type Safety

Because both the frontend and backend share TypeScript types, the client automatically knows:

* parameters
* return values
* input validation

If you change a procedure on the backend, TypeScript immediately reports errors anywhere the frontend no longer matches.

This is known as **end-to-end type safety**.

---

# How tRPC Works

A typical request looks like this:

```text
Frontend
    │
    ▼
tRPC Client
    │
 HTTP Request
    │
    ▼
HTTP Server
(Express / Hono / Fastify / Next.js)
    │
    ▼
tRPC Router
    │
    ▼
Procedure
    │
    ▼
Business Logic
```

Server:

```ts
export const appRouter = t.router({
  getUser: t.procedure.query(() => {
    return { name: "John" };
  }),
});
```

Client:

```ts
const user = await trpc.getUser.query();
```

Although it feels like you're calling a local function, the request is still sent over HTTP.

---

# Under the Hood

tRPC does **not** replace HTTP.

It simply changes how your API is designed.

Traditional REST:

```text
GET /users/1
POST /users
DELETE /users/1
```

tRPC:

```text
trpc.getUser.query()

trpc.createUser.mutate()

trpc.deleteUser.mutate()
```

Instead of exposing URLs, you expose procedures.

The HTTP requests are generated automatically by the client.

---

# tRPC is Not a Backend Framework

Although tRPC runs on the server, it is **not** a backend framework.

It does **not** provide:

* an HTTP server
* request routing
* a runtime
* database access
* file system APIs

Instead, it adds a **type-safe API layer** on top of an existing server such as:

* Express
* Fastify
* Hono
* Next.js Route Handlers
* Node's HTTP module

Because of this, tRPC cannot run by itself—it always requires another framework or server to host it.

---

# Where tRPC Fits

```text
Frontend                     Backend
─────────                    ───────────────────────
React / Next.js              Express
      │                      Fastify
      ▼                      Hono
 tRPC Client                 Next.js Routes
      │                            │
      └────── Type-Safe API ───────┘
                     │
               tRPC Router
                     │
               Business Logic
```

tRPC sits between your frontend and backend.

Its responsibility is to expose backend procedures and allow the frontend to call them with full TypeScript type safety.

---

# Frontend Integration

Many frontend features commonly associated with tRPC actually come from **TanStack Query**.

```text
React
   │
   ▼
TanStack Query
(useQuery, cache,
mutations,
optimistic updates,
SSR)
   │
   ▼
tRPC Client
   │
 HTTP
   │
Backend
```

tRPC integrates with TanStack Query so you automatically get:

* `useQuery()`
* `useMutation()`
* caching
* automatic query keys
* optimistic updates
* background refetching
* SSR support

---

# Project Structure

tRPC works best when your frontend and backend share the same TypeScript types.

In practice, this usually means using:


Sharing types removes the need for separate API schema generation.

---

# When to Use tRPC

Use tRPC when:

* both your frontend and backend are written in TypeScript
* you're building an internal API
* you want maximum developer productivity
* you want end-to-end type safety without maintaining REST or GraphQL schemas

Avoid tRPC when:

* your API will be consumed by third parties
* clients are written in other languages (Flutter, Swift, Kotlin, etc.)
* you need a language-agnostic public API

---
I'd structure **Part 2** as a hands-on tutorial that builds one application from start to finish. Every section should introduce exactly one new concept.

---

# Part 2 — Building an Application with tRPC

## 1. Project Structure

Before writing any code, organize your project so the frontend and backend can share TypeScript types.

```text
my-app/
├── apps/
│   ├── web/          # Next.js (Frontend)
│   └── backend/      # Express / Hono / Fastify
│
└── packages/
    └── trpc/         # Shared types
```

Why?

* Frontend and backend are separate applications.
* They can even run on different ports or servers.
* They only share **TypeScript types**, not backend code.

---

## 2. Installation

There are two common ways to build a tRPC application:

1. Separate backend
   - Express
   - Fastify
   - Hono

2. Meta framework
   - Next.js

The concepts are identical—the only difference is which HTTP server hosts the tRPC router.

Install only what each application needs.

### Backend

```bash
npm install @trpc/server
```

### Frontend

```bash
npm install @trpc/client
npm install @trpc/react-query
npm install @tanstack/react-query
```

The packages are deliberately separate because the backend hosts the router while the frontend only needs the client and TanStack Query integration.

---

## 3. Supported Servers (Adapters)

tRPC doesn't include its own HTTP server. Instead, it plugs into existing servers through adapters.

| Server     | Adapter                            |
| ---------- | ---------------------------------- |
| Express    | `@trpc/server/adapters/express`    |
| Fastify    | `@trpc/server/adapters/fastify`    |
| Next.js    | `@trpc/server/adapters/next`       |
| Fetch API  | `@trpc/server/adapters/fetch`      |
| AWS Lambda | `@trpc/server/adapters/aws-lambda` |
| Hono       | via Fetch adapter                  |

For this tutorial, we'll use **Express**.

---

## 4. Create the Router

Define your API using procedures.

```ts
// apps/backend/src/router.ts

import { initTRPC } from "@trpc/server"
import { z } from "zod"

const t = initTRPC.create()

export const appRouter = t.router({
  getUsers: t.procedure.query(() => {
    return [{ id: 1, name: "John" }]
  }),

  createUser: t.procedure
    .input(
      z.object({
        name: z.string(),
      })
    )
    .mutation(({ input }) => {
      return {
        id: 2,
        name: input.name,
      }
    }),
})

export type AppRouter = typeof appRouter
```

Notice that only the **type** is exported for the frontend.

---

## 5. Start the HTTP Server

Now expose the router through an HTTP server.

```ts
// apps/backend/src/index.ts

import express from "express"
import { createExpressMiddleware } from "@trpc/server/adapters/express"
import { appRouter } from "./router"

const app = express()

app.use(
  "/trpc",
  createExpressMiddleware({
    router: appRouter,
  })
)

app.listen(4000)
```

At this point, your backend is serving a tRPC API over HTTP.

---

## 6. Share the Router Type

Instead of sharing backend code, share only the router's type.

```ts
// packages/trpc/index.ts

export type { AppRouter } from "../../apps/backend/src/router"
```

The frontend imports this type to understand the API without importing any server implementation.

---

## 7. Create the tRPC Client

Configure the frontend client.

```ts
// apps/web/lib/trpc.ts

import { createTRPCReact } from "@trpc/react-query"
import { httpBatchLink } from "@trpc/client"

import type { AppRouter } from "@my-app/trpc"

export const trpc = createTRPCReact<AppRouter>()

export const trpcClient = trpc.createClient({
  links: [
    httpBatchLink({
      url: "http://localhost:4000/trpc",
    }),
  ],
})
```

This connects your frontend to the backend while preserving full type safety.

---

## 8. Add the Providers

Because tRPC integrates with TanStack Query, wrap your application with both providers.

```tsx
<QueryClientProvider client={queryClient}>
  <trpc.Provider
    client={trpcClient}
    queryClient={queryClient}
  >
    {children}
  </trpc.Provider>
</QueryClientProvider>
```

Now every React component can access tRPC.

---

## 9. Call Procedures

Queries:

```tsx
const { data, isLoading } = trpc.getUsers.useQuery()
```

Mutations:

```tsx
const createUser = trpc.createUser.useMutation()

createUser.mutate({
  name: "Jane",
})
```

Notice how:

* no URL is written
* no `fetch()`
* no request interfaces
* no response interfaces

Everything is inferred automatically.

---

## 10. What Happens Behind the Scenes?

Although it looks like you're calling local functions, the request still travels over HTTP.

```text
React Component
        │
        ▼
trpc.getUsers.useQuery()
        │
        ▼
tRPC Client
        │
        ▼
HTTP POST /trpc
        │
        ▼
Express
        │
        ▼
tRPC Router
        │
        ▼
getUsers Procedure
        │
        ▼
Database / Business Logic
        │
        ▼
Response
        │
        ▼
TanStack Query Cache
        │
        ▼
React Component
```

The only difference from REST is that tRPC generates the HTTP request and infers all the types automatically.

---

## 11. What You've Built

You now have:

* An Express backend hosting a tRPC router.
* A React/Next.js frontend using the tRPC client.
* End-to-end TypeScript type safety.
* TanStack Query handling caching, mutations, loading states, and refetching.
* A shared `AppRouter` type connecting both applications without manually writing API schemas or REST endpoints.

---

## What You Get For Free

| Feature | How |
|---|---|
| **Type-safe API calls** | `trpc.getUsers.useQuery()` — fully typed |
| **Loading states** | `isLoading` from TanStack Query |
| **Error handling** | `error` from TanStack Query |
| **Caching** | TanStack Query handles automatically |
| **Mutations** | `useMutation` with `onSuccess` callbacks |
| **Input validation** | Zod on backend, typed on frontend |
| **No API types to maintain** | Change backend = frontend breaks at compile time |

---

> tRPC basically works everywhere. The **standalone adapter** means you don't even need a framework — just raw Node.js is enough.

---

## tRPC with Plain Node.js HTTP

```ts
// Backend — plain Node.js, no framework at all
import { createServer } from 'http'
import { createHTTPHandler } from '@trpc/server/adapters/standalone'
import { initTRPC } from '@trpc/server'

const t = initTRPC.create()

const appRouter = t.router({
  getUsers: t.procedure.query(() => [{ id: 1, name: 'John' }])
})

// tRPC standalone adapter = plain Node.js HTTP
const handler = createHTTPHandler({ router: appRouter })

createServer(handler).listen(4000)
```

---

## What tRPC Actually Gives You

| Feature | tRPC | Hono | Express |
|---|---|---|---|
| **HTTP Server** | ❌ | ✅ | ✅ |
| **Routing** | ⚠️ Procedures only | ✅ | ✅ |
| **Type-safe APIs** | ✅ | ✅ | ❌ |
| **TanStack Query** | ✅ built-in | ❌ | ❌ |
| **Auth integration** | ✅ via adapters | ⚠️ manual | ⚠️ manual |
| **Standalone server** | ❌ | ✅ | ✅ |
| **Middleware** | ⚠️ limited | ✅ | ✅ |

---
tRPC gives your frontend:

✅ useQuery()
✅ useMutation()
✅ Automatic query keys
✅ Cache invalidation
✅ Prefetching
✅ SSR hydration
✅ Optimistic updates
✅ End-to-end types from the BFF to the frontend
---

**How trpc works on sperate server and frontened**

You define your API as a tRPC router in a file like `server/router.ts`. Instead of writing `app.get('/user', ...)` style routes, you write procedures — `getUser`, `createPost` — using tRPC's `t.procedure` builder. At the bottom of that file you export one thing that matters to the client:

```ts
export type AppRouter = typeof appRouter
```

This line says "take the shape of this router object and export it as a pure TypeScript type". That type knows every procedure name, every input shape, every output shape.

Then in `server/index.ts` you mount tRPC onto Express like any other middleware:

```ts
app.use(cors({ origin: 'https://your-react-app.netlify.app' }))
app.use('/trpc', createExpressMiddleware({ router: appRouter }))
```

CORS is needed because React and Express run on different origins — tRPC uses plain HTTP under the hood so the browser's CORS rules apply just like any fetch call.

---

**How the React side works**

In the client you create a bridge file `client/src/trpc.ts`:

```ts
import type { AppRouter } from '../../server/router'
export const trpc = createTRPCReact<AppRouter>()
```

The `import type` is the key. It tells TypeScript — read that server file for its type information only, import zero actual code from it. TypeScript reads `router.ts`, sees all the procedure names and shapes, and stamps them onto the `trpc` object. Then at build time that import is completely erased. Express never ships to the browser.

You then wrap your app with two providers — tRPC's provider and React Query's provider — and give tRPC the URL of your Express server:

```ts
trpc.createClient({ links: [httpBatchLink({ url: 'https://your-api.railway.app/trpc' })] })
```

Now anywhere in your app you can write `trpc.getUser.useQuery({ id: '1' })` and VS Code knows the input must be `{ id: string }` and the return type is whatever your server returns.

**How runtime requests work**

When the browser runs `trpc.getUser.useQuery()`, tRPC makes a real HTTP POST to `/trpc/getUser` on your Express server. Express receives it, tRPC's middleware routes it to the right procedure, runs it, and sends back JSON. React Query handles caching and re-fetching. The type information is completely gone by this point — it only ever existed during development and the build step.

---

**The deployment situation**

When you deploy Express to Railway and React to Netlify, the type import path becomes the problem. The file `../../server/router` doesn't exist on Netlify's servers — only your built React app lives there.

The solution almost everyone uses is a monorepo — one git repository with a `packages/server` folder and a `packages/client` folder. Railway is pointed at `packages/server` as its root. Netlify is pointed at `packages/client` as its root.

The trick is that when Netlify runs your build command, it checks out the entire repo. So `packages/server/router.ts` is sitting right there on the build machine. TypeScript finds it, reads the type, erases the import, then outputs your final JavaScript bundle — which has zero trace of Express, zero DB code, nothing from the server. That clean bundle is what gets deployed to Netlify.

Railway runs your Express server. Netlify serves your React app. The type import only ever existed during the build, on the build machine, for a few seconds.


Here's a summary table:

| Category | Feature | What It Gives You |
|---|---|---|
| **Type Safety** | Typed API calls | `trpc.getUsers.useQuery()` — fully typed, inferred from backend |
| | No manual API types | Backend changes break frontend at **compile time**, not runtime |
| | Input validation | Zod on backend → automatically typed on frontend |
| **Data Fetching** (via TanStack Query) | `useQuery()` | Fetch data with built-in loading/error/cache states |
| | `useMutation()` | Handle writes, with `onSuccess` callbacks for side effects |
| | Automatic query keys | No manual cache-key management |
| | Caching | Handled automatically by TanStack Query |
| | Background refetching | Keeps cached data fresh without extra code |
| | Optimistic updates | Update UI before server confirms, for snappier UX |
| | Prefetching | Load data ahead of navigation/render |
| | SSR hydration | Server-fetched data seamlessly hydrates on client |
| **Developer Experience** | `isLoading` / `error` | Built-in state flags, no manual state management |
| | No API client to maintain | Backend router *is* the contract — no OpenAPI/type-gen step |

**One-liner:** tRPC turns your backend router into the frontend's type-safe API client, while TanStack Query handles all the caching, loading, and mutation logic for free.