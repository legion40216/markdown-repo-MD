# The API Layer

The **API layer** is the bridge between your frontend and your backend. It defines the contract for how data is requested, sent, and received across that boundary.

---

## What Is the API Layer?

### A Better Mental Model

```text
API Layer
───────────────
REST
GraphQL
RPC

        ▲

Transport Layer
───────────────
(HTTP Framework)
Express
Fastify
Hono
Node HTTP
```

The overall web architecture looks like this:

```text
Frontend
    ↓
API Layer
    ↓
Business Logic
    ↓
Database
```

The API layer is **not your web framework**.

Frameworks such as Express, Fastify, Hono, and Next.js provide the HTTP server and routing infrastructure (the **transport layer**). REST, GraphQL, and RPC sit **on top** of that infrastructure and define **how the frontend communicates with the backend**.

There are three dominant API communication styles, each with a different mental model.

| | REST | GraphQL | RPC |
|---|---|---|---|
| **Mental model** | I expose *resources* | I expose a *graph* | I expose *functions* |
| **Example** | `GET /users/1` | `query { user(id: 1) { name } }` | `client.user.get()` |
| **Backend** | Define endpoints | Define schema + resolvers | Define routers & procedures |
| **Frontend** | `fetch()` / Axios | Apollo Client, urql | `trpc.user.get.query()` |
| **Type safety** | Manual or shared schemas | Generated from schema | Inferred from server types |
| **Key cost** | Manual networking | Schema + code generation | Shared TypeScript codebase |

---

## How Client–Server Communication Works

Before learning API Layer, it helps to understand what actually happens whenever your frontend talks to a backend. Everything builds on one simple cycle: **request and response**.

---

### The Request–Response Cycle

Every web application follows the same flow, no matter which API style it uses:

```
Frontend → HTTP Request → Internet → HTTP Server → Application Code → Database
                                                                          ↓
Frontend ← HTTP Response ←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←
```

1. The frontend needs data or wants to perform an action.
2. It sends an **HTTP request** to the server.
3. The server runs some application code and returns an **HTTP response**.

REST, GraphQL, and RPC all sit on top of this same cycle — they just organize it differently.

---

### Anatomy of an HTTP Request

An HTTP request is a message sent from the client to the server. Every request has four parts:

```
HTTP Request
├── Method
├── URL
├── Headers
└── Body (optional)
```

Here's a real example:

```http
POST /users
Authorization: Bearer eyJ...
Content-Type: application/json

{
  "name": "Suleman",
  "email": "suleman@example.com"
}
```

Annotated:

```
POST /users                      ← Method + URL
Authorization: Bearer eyJ...     ← Header
Content-Type: application/json   ← Header
                                 ← (blank line separates headers from body)
{ "name": "Suleman" }            ← Body
```

| Part        | Purpose                                              |
|-------------|------------------------------------------------------|
| **Method**  | What operation to perform (`GET`, `POST`, etc.)      |
| **URL**     | Which resource or endpoint to access                 |
| **Headers** | Metadata: auth tokens, content type, cookies, etc.   |
| **Body**    | The data being sent (optional)                       |

#### HTTP Methods

| Method     | Meaning                  |
|------------|--------------------------|
| `GET`      | Retrieve data            |
| `POST`     | Send new data            |
| `PUT`      | Replace existing data    |
| `PATCH`    | Partially update data    |
| `DELETE`   | Remove data              |

> **Note:** `GET` requests usually have no body — you're only asking for data, not sending any.

---

### What Happens on the Server?

When the request arrives, an HTTP framework (Express, Fastify, Hono, etc.) takes over. Its job is to figure out: *which piece of code should handle this?*

This process is called **routing**.

```
Incoming Request → HTTP Framework → Route Matching → Middleware → Handler → Business Logic → Database
```

For example:

```js
app.post("/users", createUser)
```

When `POST /users` arrives, the framework finds this route and runs `createUser(req, res)`. Think of the framework as a traffic controller — it directs each request to the right function.

---

## API Communication

Modern web applications generally use one of three API styles:

- REST
- GraphQL
- RPC

These have different ways of organizing communication between a client and a server. But regardless of which one you use, they all run on the same HTTP as the transport layer.

- Headers for metadata (auth, content type, cookies)
- JSON for data payloads
- Standard authentication patterns (JWT, sessions, cookies)

---


### The Three API Communication Styles

````markdown
## REST — Resource-Based Communication

REST (**Representational State Transfer**) models your application around **resources**. Each resource is identified by a URL, and clients interact with those resources using standard HTTP methods.

```text
/users
/users/123
/orders
/products/42
```

For example:

```http
GET    /users/1
POST   /users
PUT    /users/1
DELETE /users/1
```

REST is the most widely adopted API style and forms the foundation of many public web APIs. Its biggest strengths are **simplicity**, **predictability**, and **universal compatibility**.

---

### HTTP Methods

| Method | Purpose |
| ------ | ------- |
| **GET** | Retrieve a resource |
| **POST** | Create a resource |
| **PUT / PATCH** | Update a resource |
| **DELETE** | Remove a resource |

---

### Examples

#### Retrieve a User

```http
GET /users/123
Authorization: Bearer <token>
```

#### Create a User

```http
POST /users
Content-Type: application/json

{
  "name": "Suleman"
}
```

---

### What You Manage in REST

When building a REST API, you're responsible for designing and maintaining:

- URL structure
- HTTP methods for each endpoint
- Request and response validation
- Client-side `fetch()` (or another HTTP client) calls
- API documentation
````

---

#### GraphQL — Query-Based Communication

GraphQL models your API as a graph of data rather than a fixed set of endpoints. Instead of many URLs returning predetermined shapes, there is usually a single endpoint:

POST /graphql

The client sends a query describing exactly which fields it wants — no more, no less.

graphqlquery {
  user(id: "123") {
    name
    posts {
      title
    }
  }
}

The schema defines everything clients are allowed to query or mutate, acting as a contract between frontend and backend.

This avoids:

- Over-fetching (receiving unnecessary data)
- Under-fetching (making multiple requests)

The tradeoff is additional complexity through schemas, resolvers, and tooling.

---

#### RPC — Function-Based Communication

RPC **(Remote Procedure Call)** models your API as remote functions.

Instead of exposing resources identified by URLs, the server exposes procedures.

- trpc router
```
createUser()
deleteUser()
resetPassword()
```

The frontend calls backend procedures almost as if they were local functions.

- trpc client  
```
await client.user.create({
  name: "Suleman"
})
```

The RPC client handles constructing a request by choosing a URL, an HTTP method, and formatting a request body those details automatically.

Notice there is no URL, no HTTP method, no fetch() and formatting a request body. You're just calling a function.

Everything revolves around the router.

RPC is particularly popular in full-stack TypeScript applications.

---

## Type-Safe APIs: The Real Differentiator

As TypeScript has become the standard for full-stack development, one of the biggest improvements in developer experience has been **type-safe communication between the frontend and backend**.

The goal is simple:

> **When the server changes, the client should know immediately—before your application runs.**

Without this guarantee, the frontend and backend can silently drift apart, leading to runtime bugs that TypeScript cannot detect.

---

### Benefits of Type-Safe APIs

Type-safe APIs help you:

- Catch request and response mismatches at compile time.
- Reduce runtime bugs caused by stale API contracts.
- Improve refactoring by propagating API changes automatically.
- Keep frontend and backend synchronized.

---

### Type Safety Has Nothing to Do with Where Your Server Runs

Type safety is **not** determined by deployment.

It doesn't matter whether your backend is:

- Express
- Fastify
- Hono
- Next.js
- Running locally
- Running on another server

What matters is **how both sides agree on the API contract**.

For example:

```text
my-app/
├── frontend/
├── backend/
└── packages/
    └── types/
```

The frontend and backend can communicate safely even while running on different ports or machines.

---

### TypeScript Support ≠ Type-Safe APIs

These two ideas are often confused.

#### TypeScript Support

TypeScript support simply means your backend is written in TypeScript.

```ts
app.get("/users", async () => {
  return [{ name: "John" }]
})
```

This provides type checking **inside the backend**.

It says nothing about whether the frontend knows the response shape.

For example:

```ts
const res = await fetch("/api/users")

const users = (await res.json()) as User[]
```

The `as User[]` is only a type assertion.

If the backend changes, TypeScript will not warn you.

The application may only fail once it runs.

---

#### Type-Safe APIs

Type-safe APIs solve this problem by ensuring both sides agree on the same API contract.

Different API styles achieve this in different ways.

##### 1. Shared Runtime Schemas (REST)

In a normal REST setup, the frontend and backend are completely disconnected. The backend developer builds GET /api/v1/products, and the frontend developer has to look at documentation (like Swagger/OpenAPI) to know what data that endpoint returns

A common solution to REST approach is to define a runtime schema once using Zod and share it between the server and client.

```ts
export const UserSchema = z.object({
  name: z.string(),
  email: z.string().email(),
})

export type User = z.infer<typeof UserSchema>
```

Server:

```ts
return Response.json(
  UserSchema.parse({
    name: "John",
    email: "john@example.com",
  })
)
```

Client:

```ts
const user = UserSchema.parse(await res.json())
```

This provides:

- Runtime validation
- Shared TypeScript types

However, you still manually write:

- `fetch()`
- URLs
- HTTP methods
- Request handling

The API contract is shared, but the networking remains manual.

---

##### 3. RPC

RPC libraries such as tRPC take a different approach.

Instead of generating types from a schema, they infer types directly from your server procedures.

```ts
export const appRouter = router({
  user: ...
})

export type AppRouter = typeof appRouter
```

Client:

```ts
const user = await client.user.get.query()
```

TypeScript automatically infers:

- Inputs
- Outputs
- Procedure names

The tradeoff is that this approach typically assumes both frontend and backend are written in TypeScript and can share types, usually through a monorepo or a shared package.

---

### Comparison

| Approach | How Type Safety Works | Build Step | Runtime Validation |
|-----------|-----------------------|------------|--------------------|
| RPC (tRPC, Hono RPC) | Shared router types | ❌ | Optional |
| GraphQL | Schema → Code Generation | ✅ | Yes |
| OpenAPI | Specification → Code Generation | ✅ | Depends |
| Shared Zod + REST | Shared runtime schemas | ❌ | ✅ |
| Plain REST + `fetch()` | Manual typing | ❌ | ❌ |

---

### Key Takeaway

- **Plain REST** relies on manual typing.
- **Shared Zod** shares runtime schemas between client and server.
- **GraphQL** generates TypeScript types from a schema.
- **RPC** infers types directly from server code.

Although these approaches differ in implementation, they all aim to solve the same problem:

> **Keeping the frontend and backend synchronized so API changes are caught during development instead of at runtime.**


### RPC Frameworks

RPC frameworks generally fall into two categories:

```text
RPC Frameworks
│
├── Framework-agnostic
│   │
│   ├── tRPC
│   │    ├── Express
│   │    ├── Fastify
│   │    ├── Hono
│   │    └── Next.js
│   │
│   ├── oRPC
│   │    ├── Express
│   │    ├── Fastify
│   │    ├── Hono
│   │    ├── Elysia
│   │    └── Next.js
│   │
│   └── gRPC
│        └── Multiple languages
│
└── HTTP Framework-specific
    │
    ├── Hono RPC
    │    └── Requires Hono
    │
    └── Eden Treaty
         └── Requires Elysia
```

there are servers that have built in rpc like Http framework rpc  = server AND can do RPC,  Http framework rpc can stand alone. 
RPC frameworks cannot exist without a server underneath it.

RPC is
```
✅ Auto React Query hooks
✅ Server caller for authenticated Server Components
✅ No codegen
✅ TypeScript native
✅ Works inside Next.js
```
Http framework rpc

## Key Difference vs Framework-agnostic RPC

| | Framework-agnostic RPC  | Hono RPC | Eden |
|---|---|---|---|
| **Is a framework** | ❌ | ✅ | ❌ |
| **Needs host server** | ✅ | ❌ | ❌ |
| **REST compatible** | ❌ | ✅ | ✅ |
| **Compile time safe** | ✅ | ✅ | ✅ |
| **TanStack Query** | ✅ built-in | ⚠️ manual | ⚠️ manual |
| **Edge runtime** | ⚠️ depends on host | ✅ | ❌ |

---

#### Framework-agnostic RPC

These RPC frameworks can work with multiple HTTP frameworks by using adapters.

Examples:
- tRPC
- oRPC

## Type-Safe RPC Systems

### tRPC

"I have TypeScript on both the frontend and backend. I don't want to manually define REST endpoints or API schemas."

#### Why tRPC Exists

REST, GraphQL, and RPC let a client talk to a server, but none of them keep your frontend and backend types in sync automatically. If the backend changes — a field gets renamed, say — the frontend doesn't know. It only breaks later, at runtime.

tRPC fixes this for full-stack TypeScript apps: frontend and backend share the same types directly (usually via a monorepo), so a backend change shows up as a type error on the frontend immediately,

> **Important:** The type safety does **not** come from running the frontend and backend on the same server. It comes from **sharing TypeScript types** between them.

---

#### The Core Idea

Normally, a backend API involves two separate pieces:

- The **handler function** on the server — the actual code that runs when a request comes in (e.g. an Express route like app.post('/users', handler))
- A **type definition** (e.g For example:
    ```
    typescripttype User = {
      id: number;
      name: string;
      email: string;
    };
    ```
  ) 
on the frontend, written separately, describing what that endpoint expects and returns

Because these are two separate things, nothing keeps them in sync. If the backend changes, the frontend's types don't update automatically — the mismatch only shows up later, when the app breaks at runtime.
tRPC removes this duplication. Instead of writing a separate type by hand, the backend handler function itself becomes the source of truth. TypeScript automatically infers the request/response types directly from that function — so the frontend always matches what the backend actually does, and your editor will flag a mismatch before you even run the code.

---

#### One Procedure, One Source of Truth

Backend:

```ts
export const appRouter = t.router({
  getUser: t.procedure.query(() => ({
    userId: 1,
    fullName: "John",
  })),
})

export type AppRouter = typeof appRouter
```

This single definition serves two purposes:

- It is the runtime code executed on the server.
- It is also the compile-time API contract used by the frontend.

There is no separate interface or schema to keep in sync.

---

#### Calling Procedures from the Frontend

```ts
const { data } = trpc.getUser.useQuery()
```

This looks like an ordinary TypeScript function call.

There are:

- No URLs to write
- No `fetch()` calls
- No response interfaces to maintain manually

TypeScript already knows everything about the procedure.

```ts
data.userId
data.fullName
```

The frontend automatically knows:

- Procedure names
- Input types
- Return types

---

#### Compile-Time Safety

Suppose your backend originally returns:

```ts
getUser: t.procedure.query(() => ({
  id: 1,
  name: "John",
}))
```

Later, you rename the fields:

```ts
getUser: t.procedure.query(() => ({
  userId: 1,
  fullName: "John",
}))
```

Your existing frontend code immediately breaks:

```ts
const { data } = trpc.getUser.useQuery()

console.log(data.id)       // ❌ Compile error
console.log(data.name)     // ❌ Compile error

console.log(data.userId)   // ✅
console.log(data.fullName) // ✅
```

Because the frontend derives its types directly from the backend, TypeScript immediately reports every place that needs updating.

You can't successfully build the application until those errors are fixed.

---

#### What Actually Happens?

Calling a procedure:

```ts
await trpc.user.getById.useQuery({
  id: "123",
})
```

Feels like calling a local function.

Behind the scenes, the tRPC client converts it into an ordinary HTTP request.

```http
POST /api/trpc/user.getById
Content-Type: application/json
```

```json
{
  "input": {
    "id": "123"
  }
}
```

The server executes the procedure, returns a JSON response, and the client converts it back into the value returned by `useQuery()`.

tRPC doesn't eliminate HTTP—it simply hides the networking behind a strongly typed client API.

---

#### Authentication

Authentication works exactly the same way as in any other HTTP application.

For example:

```ts
export async function createContext({ req }) {
  const token = req.headers.authorization
  const user = verifyToken(token)

  return { user }
}
```

You can use:

- JWTs
- Cookies
- Sessions
- Custom authentication

Because the browser is still sending ordinary HTTP requests, all standard authentication techniques continue to work.

---

#### Why Developers Like tRPC

Instead of manually writing:

- URLs
- HTTP methods
- `fetch()` calls
- Request types
- Response types

You simply call backend procedures.

This gives you:

- Automatic autocomplete
- Compile-time type checking
- Safer refactoring
- Less duplicated code
- A single source of truth for your API

---

#### Key Takeaway

tRPC is a **type-safe RPC framework for TypeScript**.

Its defining feature is that **the backend procedures are the API contract**. Rather than maintaining separate client types, TypeScript infers everything directly from the server.

The result is an API that feels like calling local functions while still communicating over ordinary HTTP behind the scenes.


### 4. oRPC — the newest one
oRPC is a TypeScript library that includes first-class OpenAPI support, end-to-end type safety for inputs, outputs, and errors, native support for complex types such as Date and File, and seamless integration with popular frontend frameworks including React, Vue, Solid, and Svelte through TanStack Query.

The unique thing about oRPC is it bridges RPC **and** OpenAPI together:
```ts
const getUsers = os.route({
  method: 'GET',
  path: '/users',
}).handler(() => [{ id: 1, name: 'John' }])
```
✅ Compile-time safe · ✅ TanStack Query · ✅ OpenAPI spec generated automatically · ✅ Not a framework (like tRPC)

---

### 5. ts-rest
ts-rest is another contender in the modern TypeScript ecosystem for building typesafe APIs with its own philosophy and tradeoffs. Contract-first approach — you define the contract first, then implement it:
```ts
// Define contract first
const contract = c.router({
  getUsers: { method: 'GET', path: '/users', responses: { 200: z.array(UserSchema) } }
})

// Backend implements it
// Frontend consumes it
// Both bound to same contract
```
✅ REST style · ✅ Compile-time safe · ✅ Works with Express/Fastify/Next.js

#### HTTP Framework RPC
These RPC implementations are tightly coupled to a particular HTTP framework and cannot be used independently of it.

Examples:

- Hono RPC (requires Hono)
- Eden Treaty (requires Elysia)

--
server frameworks with rpc 

HTTP framework such as hono

begins with:

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

RPC came later.

---

Hono RPC is an extra feature

After defining your normal routes:

```ts
app.get("/users", ...)
app.post("/users", ...)
```

Hono can generate a typed client from those routes.

```ts
const client = hc<AppType>("https://api.example.com")

await client.users.$get()
```

Now you get autocomplete.

So Hono says:

> "Since we already know all your routes, why not generate a typed client automatically?"

Hono RPC

```ts
client.users.$get()
```

literally performs

```
GET /users
```

The client is just giving you type safety over standard HTTP routes.

Hono RPC gives you:

- Standard REST endpoints
- Works with any client that can make HTTP requests
- Type-safe TypeScript client for your own frontend
- No separate RPC framework to learn

It feels like:

> "REST API + automatic TypeScript client."

HTTP Framework-specific RPC  is a **full backend framework** that ALSO has RPC built in — so you get the best of both worlds. Unlike RPC which needs a host, HTTP Framework-specific RPC  IS the server AND provides compile-time type-safe RPC. The only thing it lacks vs tRPC is the **built-in TanStack Query integration**.

| Framework | Type-Safe | Edge | Nextjs Wrapper
|---|---|---|---|---|---|

| **Hono RPC** | ✅  ✅ | ✅ |
| **Elysia Eden** | ✅  ✅ | ❌ |

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

## 3. Authentication Context Must Be Managed Normally

Authentication follows normal web API patterns.

Requests may require:

- Cookies
- Authorization headers
- Bearer tokens
- Sessions

Hono RPC does not automatically infer or forward authentication information between different HTTP requests.

---

## Elysia + Eden

**Eden** is Elysia's official end-to-end type-safe client.

Like Hono RPC, Eden infers the types of your Elysia routes and provides a fully typed client without code generation.

```ts
client.users.get()
client.users.post({ name: "John" })
```

Although the API feels like RPC, it still communicates using standard HTTP. The only thing being shared is the TypeScript types.

**Summary**
✅ Bun optimized  
✅ End-to-end type safety  

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

## 3. Authentication Works Like Any Other HTTP API

Authentication follows normal web API patterns.

Requests may require:

- Cookies
- Authorization headers
- Bearer tokens
- Sessions

Eden does not automatically forward authentication information between separate HTTP requests.

---

| HTTTP Framework RPC | data-fetching libaray | Edge | Nextjs(Wrapper)
|----------|-------|----------|
| Hono rpc |  ✅ |  ✅  | ✅ 
| Elysia + Eden | ✅ |  | ✅

### RPC Implementation Patterns

See all RPC frameworks at [RPC Frameworks](#rpc-frameworks).

---

#### Meta Framework

##### RPC framework Implementation

The RPC framework integrates directly with the meta framework.

```text
Next.js (Meta Framework)
├── React
├── App Router
└── RPC Framework
```

**Examples**
- tRPC
- oRPC

---

##### HTTP Framework-specific RPC Implementation

The RPC implementation is built into a specific HTTP framework.

Requirements:
- Requires an HTTP framework (e.g. Hono, Elysia).
- Requires an adapter to integrate the HTTP framework into a meta framework such as Next.js.
- The RPC implementation only works with its own HTTP framework.

```text
Next.js (Meta Framework)
├── React
└── HTTP Framework
    ├── Hono
    │   └── Hono RPC
    └── Elysia
        └── Eden Treaty
```

**Examples**

Edge-compatible
- Hono + Hono RPC

Node.js only
- Elysia + Eden Treaty

---

#### Standalone HTTP Framework

The frontend communicates with a standalone RPC server.

```text
Frontend
└── React
    └── RPC Client

Backend
└── HTTP Framework
    └── RPC Server
```

**Examples**
- Express + tRPC
- Hono + Hono RPC
- Elysia + Eden Treaty

---

### Choosing an Architecture

**Option 1: Meta-Framework + RPC Frameworks** (Edge compatiable)
```
Meta-framework (frontend + API routes)
        │
RPC Framework (tRPC, oRPC)
        │
Business Logic
```
Best for monorepos and full-stack TypeScript apps where you want maximum type safety with minimal setup.

**Option 2: Meta-Framework + Http Framework + Http Framework RPC** (Edge compatiable) - depends on Http Framework
```
Meta-framework (frontend + API routes)
        │
HTTP Framework (Hono, Elysia, etc.)
        │
Business Logic
        ▲
        │
HTTP Framework RPC Client (generated from the HTTP framework)
```

**Option 3: Meta-Framework, Http Framework + Http Framework RPC** (Edge compatiable) - depends on Http Framework
```
Frontend (Next.js) (localhost:3000)
        │
        ▼
tRPC (inside Next.js)
        │
        ▼
Hono API (localhost:3001)
        │
        ▼
Business Logic
```
tRPC acts as a proxy/BFF (Backend for Frontend) that calls the Hono API.

---

## The mental model

```
Browser/App
    ↕  (RPC client / Apollo Client / fetch)
API Layer   ← GraphQL & RPC both live here
    ↕  (resolvers / procedures)
Database / Services
```
Both replace the need to manually write `fetch('/api/users')` and manage types yourself — that's the core value they share.

 while **GraphQL is more of a standard** that you build around with the ecosystem. But yes, same layer, same purpose.

## Summary

- The **API layer** defines how your frontend communicates with your backend.
- The three major styles are **REST** (resources), **GraphQL** (graphs), and **RPC** (functions).
- HTTP frameworks like Express and Hono are **transport layers** — they can host any API style.
- **Type safety** is the key driver in modern API choices. RPC provides it automatically; other approaches require schemas or codegen.
- **tRPC** is popular because it's framework-agnostic and delivers end-to-end type safety with zero extra tooling.
- **Hono RPC** and **Eden** are excellent but ecosystem-specific — only use them if you're already in the Hono or Elysia ecosystem.

---

## Notes

> **Framework agnostic**
> **HTTP frameworks ≠ API styles.** Tools like Express, Hono, and Fastify are just HTTP server frameworks — they handle requests and responses. What you build on top of them (REST, GraphQL, RPC) is a separate design choice.

The three main API styles each have a distinct mental model:

- **REST** — you expose *resources* (e.g., `/users`, `/orders`)
- **GraphQL** — you expose a *graph of data* (e.g., `query { users { posts } }`)
- **RPC** — you expose *functions* (e.g., `user.getAll()`)

Express got associated with REST simply because most people historically used it that way — but it was never REST-specific.

---

