# 🔐 Auth in Web Development — Knowledge Node

## The Core Problem — HTTP is Stateless

Every HTTP request carries **no memory of previous requests**. Each one is
a blank slate — the server doesn't know who made the last request, what
they did, or any prior context. Every request must be self-contained.

> Authentication exists to solve one problem:
> *"How does the server know who is making this request?"*

### How state is layered on HTTP requests

| Mechanism | How it works | Manual attachment? |
|---|---|---|
| Cookies | Browser sends automatically on same-origin requests | ❌ No |
| JWT / Auth headers | Client attaches a token per request | ✅ Yes |
| Sessions | Server stores state; client holds only a session ID | ✅ Yes |
| Query params | State encoded in the URL | ✅ Yes |

Cookies are the **one exception** — the only mechanism the browser handles
for you. Everything else must be stitched into each request manually.

---

## Authentication — The Abstracted View
```
┌─────────────────────────────────────────────────────┐
│                   THE CHAIN                         │
│                                                     │
│  User logs in → Server verifies → Server needs      │
│  to REMEMBER this for future requests               │
│                                                     │
│  HOW it remembers = Auth methods│
└─────────────────────────────────────────────────────┘
```

Every auth system, regardless of implementation, answers the same two questions:

**1. What credential proves your identity?**
Could be a session ID, a JWT, an API key, an OAuth token, a magic link token, or something else entirely. The format varies, but the purpose is the same: a secret the server can verify.

**2. How does that credential travel with each request?**
Broadly, two paths:
- **Automatic** — the browser handles it for you (cookies)
- **Manual** — your code attaches it to each request (Authorization header, query param, etc.)

These are independent choices. You mix and match: any credential format can ride in a cookie *or* a header.

| Mechanism | Managed by | Transport |
|---|---|---|
| Cookies | Browser (automatic) | Same-origin requests |
| Authorization header | Your code (manual) | Any origin |
| Query params | Your code (manual) | Any origin |

---

## The Two Most Common Formats

### JWT
A signed, self-contained token that encodes identity and claims directly inside it. The server verifies the signature — no database lookup needed.

```
token = encoded user data + signature → cookie/localStorage holds it
server just verifies the signature, no DB lookup needed
```

### Session ID
An opaque reference that points to data stored on the server (DB or cache). Lightweight on the client; the server does the heavy lifting.

```
token = random string → stored in DB → cookie holds the token
server checks DB on every request to validate
```

| | Session/DB | JWT |
|---|---|---|
| State lives | Server (DB or cache) | Client (inside token) |
| Revocation | ✅ Instant (delete from DB) | ❌ Hard (valid until expiry) |
| Scalability | ⚠️ Shared session store needed | ✅ Stateless, works across servers |
| Token size | Small (just an ID) | Larger (carries claims) |
| Best for | Monoliths, admin apps | APIs, microservices, SPAs |

---

## Auth taxonomy

Everything branches from one idea: *prove who you are, then remember it.*

**Sessions** — server owns the state
- `DB session` — token maps to a row in the database
- `In-memory` — token maps to server RAM (e.g. `express-session`)

**Tokens** — client carries the state
- `JWT` — signed JSON, self-contained claims
- `Paseto` — JWT with better cryptographic defaults
- `Opaque token` — random string the server validates (functionally = DB session)

**Third-party** — delegate auth entirely
- `Auth.js` / `Better Auth` — self-hosted abstraction over sessions + tokens
- `Clerk` / `Auth0` / `Kinde` — fully hosted, you own nothing


## Transport Tradeoffs

The transport choice is independent of format — and it's where most of the security tradeoffs live:

| | Cookie | Manual (header/etc.) |
|---|---|---|
| Developer effort | Low — browser handles it | High — you wire every request |
| XSS exposure | Low (`HttpOnly` blocks JS) | Higher (JS must read it) |
| CSRF exposure | ⚠️ Needs mitigation | Not applicable |
| Cross-origin | ⚠️ Needs `SameSite` config | ✅ Easier |

> **Most secure common setup:** JWT (or session ID) in an `HttpOnly` cookie — you get tamper-proof signing *and* XSS protection.

---

The key mental model: you're always making **two independent decisions** — *what* the credential is, and *how* it travels.

## What This Looks Like in Practice

A bare fetch is anonymous by default:

```js
fetch('[https://api.example.com/data](https://api.example.com/data)')
// server sees a stranger — no identity, no context, no permissions

```

To give the server context, you have two paths:

**Cookie-based — browser handles it for you:**

```js
fetch('[https://api.example.com/data](https://api.example.com/data)', {
  credentials: 'include'  // browser auto-attaches the cookie
})

```

**Header-based — you attach it manually:**

```js
fetch('[https://api.example.com/data](https://api.example.com/data)', {
  headers: {
    'Authorization': `Bearer ${userToken}`,  // read from localStorage by your code
  }
})

```

> `credentials: 'include'` is easy to miss — by default, `fetch` strips cookies
> on cross-origin requests, so you have to opt in explicitly.

## Cookies
A cookie is just the browser **automatically attaching a small string** to every 
request to that domain. Nothing more — it's the transport layer.

--- STORING (Server → Browser) ---
Server sends in response header:
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax; Max-Age=86400

**Cookies are established by the server** (not the client).
The server generates the value, the browser just obeys and saves it.

Exception: client can set cookies via JS → document.cookie = "theme=dark"
But ONLY for non-sensitive data (themes, preferences) — never for auth/session
because JS-set cookies have no HttpOnly flag and are vulnerable to XSS theft.

--- SENDING (Browser → Server) ---
fetch('/api/plan', { credentials: 'include' })
↑ tells the browser to attach the saved cookie to the request

Same-site  (myapp.com → myapp.com/api)      → cookie sends automatically ✅
Cross-origin (myapp.com → api.myapp.com)    → MUST add credentials: 'include' ✅

For cross-origin, server must also respond with:
  Access-Control-Allow-Credentials: true
  Access-Control-Allow-Origin: https://myapp.com  ← must be exact, not *

--- Security flags ---
| Flag          | What it does                          |
|---------------|---------------------------------------|
| HttpOnly      | JS cannot read cookie → blocks XSS    |
| Secure        | Only sent over HTTPS                  |
| SameSite=Lax  | Blocks cross-site CSRF attacks        |



## The Multi-Step Journey Problem

In a login → plan → checkout flow, each fetch is independent — the server
remembers nothing. With **cookie-based auth** the browser re-attaches your
credential automatically. With **header-based auth** you are responsible for
attaching it to every request — and nothing warns you if you forget.

Step 1: POST /login → server returns a credential  
Step 2: GET /plan → credential must be present, or server returns 401  
Step 3: POST /checkout → credential must be present again  

Forgetting it at any step gets you a `401 Unauthorized`, a guest-level
response, or an anonymous result — the server has no way to distinguish
intentional omission from a bug.

**Cookie-based — browser handles it, nothing to attach:**
```js
// Step 1: POST /login → server sets a cookie in the response
// Step 2, 3: browser sends that cookie automatically on every request
fetch('/api/plan',     { credentials: 'include' })  // ✅ cookie goes along for free
fetch('/api/checkout', { credentials: 'include' })  // ✅ cookie goes along for free
```

**Header-based — you carry and attach it manually:**
```js
// Step 1: POST /login → server returns a token in the response body
const { token } = await loginResponse.json()

// Step 2, 3: YOU are responsible for attaching it every time
fetch('/api/plan',     { headers: { Authorization: `Bearer ${token}` } })
fetch('/api/checkout', { headers: { Authorization: `Bearer ${token}` } })
```

> The "forgetting the token" bug is a **header-based** problem. With cookies
> you can't really forget — the browser just does it. With manual attachment,
> a missing header silently looks anonymous to the server.

---

### In Next.js

This applies to Route Handlers, Server Actions, and Server Components
equally — each invocation is a fresh request:

```js
// Auth state is never "remembered" — you must re-read it every time
const session = await getServerSession()
```

If auth is cookie-based, the browser handles attachment automatically:

```js
fetch('/api/protected') // cookie sent automatically — no manual header needed
```

Otherwise, attach tokens yourself on every call.

## The Mental Model

```
Cookie  = the envelope       → how data travels browser ↔ server automatically
Session = post-it on server  → server remembers you by an ID (stateful)
JWT     = a signed ID card   → server trusts the card without looking anything up (stateless)
```

---

## Browser Storage & Auth

When you open **DevTools → Application → Storage**, you'll find three storage mechanisms. They look similar but behave very differently.

---

### 1. Cookies — Where session IDs and tokens live

The cookie itself holds almost nothing useful — typically just a random **Session ID** like `s:3j9F...`. The actual user data (who you are, your cart, your permissions) lives on the **server**, which uses that ID to look you up.

Modern apps also store **JWT tokens** in cookies, specifically with the `HttpOnly` flag. This flag blocks JavaScript from touching the cookie at all, which protects it from being stolen by malicious scripts.

---

### 2. Local Storage — Where tokens *also* live

Apps using pure JWT auth (no cookies) often just drop the token straight into Local Storage under a key like `token` or `access_token`.

The key difference from cookies: **it never expires on its own.** It sits there until the user logs out or manually clears their browser storage.

---

### 3. Session Storage — The imposter

This is the confusing one. The name *sounds* like server-side sessions, but it has nothing to do with them.

Session Storage is just a **temporary browser-side closet**. It dies the moment you close the tab. Because of that, almost no one uses it for auth — it's mainly used for throwaway UI state, like tracking which step of a multi-step form you were on before you hit refresh.

---

**Quick mental model:**

| Storage | Lifespan | Accessible by JS? | Typical use |
|---|---|---|---|
| Cookie (`HttpOnly`) | Until expiry / logout | ❌ No | Secure tokens, session IDs |
| Local Storage | Forever | ✅ Yes | JWTs in simpler apps |
| Session Storage | Until tab closes | ✅ Yes | Temporary UI state |

Go ahead and inspect a few websites you use daily (like GitHub, Netflix, or Reddit). Do you see mostly JWT tokens sitting in their Local Storage, or are they using classic Cookies?

---

## How the Server Verifies a Cookie

The browser sends the cookie automatically — but the server still has to **read it, extract the value, and decide if it trusts it**. Here's exactly what happens on every incoming request:

### Step-by-step

```
1. Browser makes a request
   GET /dashboard
   Cookie: session=abc123   ← browser attached this automatically

2. Server reads the cookie header
   const raw = request.headers.get('cookie')
   // "session=abc123"

3. Server extracts the value
   const token = cookies().get('session')?.value
   // "abc123"

4. Server decides what to trust based on auth strategy:
   → DB Session:  look up "abc123" in the sessions table
   → JWT:         verify the cryptographic signature of "abc123"

5. Result:
   → Found / valid   = user is authenticated, continue
   → Not found / bad = reject, redirect to /login
```

### Strategy A — DB Session verification

The token is just a random string. The server proves identity by finding it in the database:

```js
const token = cookies().get('session')?.value
if (!token) redirect('/login')

// The DB is the source of truth
const session = await db.session.findUnique({ where: { token } })

if (!session)                        redirect('/login')  // token doesn't exist
if (session.expiresAt < new Date())  redirect('/login')  // token expired

// Passed — we know who this is
const userId = session.userId
```

The cookie value means nothing on its own. It only works if a matching row exists in the DB.

### Strategy B — JWT verification

The token is self-contained. The server proves identity by verifying the cryptographic signature — no DB needed:

```ts
const token = cookies().get('session')?.value
if (!token) redirect('/login')

try {
  const { payload } = await jwtVerify(token, SECRET)
  // payload = { userId: '123', role: 'admin', exp: 1234567890 }
  // if the signature is valid, we trust what's inside
  const userId = payload.userId
} catch {
  redirect('/login')  // signature invalid or token expired
}
```

The cookie value is the proof itself — the server just checks the math.

## Sessions (Stateful, Server-Side)

Server stores who you are in a database, gives you a **random session ID**. That ID lives in a cookie. Server looks it up on every request.

```
LOGIN:
  you → server (email + password)
  server saves session row in DB
  server sends back session_id in cookie

EACH REQUEST:
  you → server (cookie: session_id=abc123)
  server queries DB: SELECT * FROM sessions WHERE token = 'abc123'
  server knows who you are
```

### DB Session flow (minimal Next.js / Express)

```js
// REGISTER
const hash = await bcrypt.hash(password, 12)
await db.user.create({ data: { email, password: hash } })

// LOGIN
const user = await db.user.findUnique({ where: { email } })
const match = await bcrypt.compare(password, user.password)
if (!match) return error('Invalid credentials') // same message always — no enumeration

const token = crypto.randomBytes(32).toString('hex') // random, not JWT
await db.session.create({ data: { userId: user.id, token, expiresAt: in7Days } })
// set token in httpOnly cookie

// AUTH CHECK
const token = cookies().get('session')?.value
const session = await db.session.findUnique({ where: { token } })
if (!session || session.expiresAt < new Date()) redirect('/login')

// LOGOUT — token is INSTANTLY dead
await db.session.delete({ where: { token } })
```

### Prisma schema

```prisma
model User {
  id       String    @id @default(cuid())
  email    String    @unique
  password String    // always hashed — never plain text
  sessions Session[]
}

model Session {
  id        String   @id @default(cuid())
  token     String   @unique
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  expiresAt DateTime
}
```

### Why DB sessions win for most projects

- Instant revocation → delete the row, token is dead immediately
- "Log out all devices" → `db.session.deleteMany({ where: { userId } })`
- Ban a user → delete all their sessions, they're out right now
- Easy to reason about and debug
- No extra library needed (`crypto` is built into Node.js)

---

# JWT (JSON Web Token)

Instead of the server remembering who you are (stateful), it hands you a **signed token** after login. Every future request carries that token — the server just verifies the signature, no DB lookup needed.

## How it works

1. You send credentials → server verifies them → server generates a signed JWT → sends it back
2. Frontend stores the token (memory, local storage, or a secure cookie)
3. Every request attaches it: `Authorization: Bearer <token>`
4. Server decodes and verifies the signature — no session store needed

## Why JWT exists

Sessions require a shared store (Redis or DB) that every server can reach. That's painful when you scale horizontally. JWT is stateless — any server can verify a token independently, with no coordination.

## When JWT fits well

| Scenario | Why it fits |
|----------|------------|
| Multiple servers / microservices | Any server verifies the token independently |
| SPAs (React, Vue, etc.) | `Authorization` header works cleanly with REST and GraphQL APIs |
| Mobile apps | Native apps can't use cookies easily — headers are natural |
| Cross-domain APIs | No CORS cookie complications |

### How it works (flow)
```
1. User logs in (email + password) → POST /login

2. Backend verifies credentials → generates a JWT and sends it back

3. Frontend stores the JWT (localStorage or cookie)

4. Every future request → frontend sends JWT in the header:
   Authorization: Bearer <token>

5. Backend verifies the token signature → knows who you are → allows/denies
```

JWT (JSON Web Token) is part of **authentication** — which is a backend responsibility, but JWT itself touches both frontend and backend.

**The core idea:**

Instead of the server storing your session in a database, it gives you a **signed token** that *you* carry around. Every request you make, you attach that token, and the server trusts it because it can verify the signature.

---

**How it works (flow):**

```
1. User logs in (email + password) → POST /login

2. Backend verifies credentials → generates a JWT and sends it back

3. Frontend stores the JWT (localStorage or cookie)

4. Every future request → frontend sends JWT in the header:
   Authorization: Bearer <token>

5. Backend verifies the token signature → knows who you are → allows/denies
```

---

**What's inside a JWT:**

A JWT is just 3 base64 parts separated by dots:

```
header.payload.signature
```

- **Header** → algorithm used (HS256 etc.)
- **Payload** → your data (userId, role, expiry) — *not encrypted, just encoded*
- **Signature** → server signs it with a secret key to prevent tampering

---

**Key points to understand:**

- The payload is **readable by anyone** — never put passwords or sensitive data in it
- But it **can't be faked** — without the secret key, the signature won't match
- JWTs are **stateless** — the server doesn't store anything, it just verifies

---

### JWT flow (Next.js with `jose`)

```ts
import { SignJWT, jwtVerify } from 'jose'
const SECRET = new TextEncoder().encode(process.env.JWT_SECRET)

// CREATE TOKEN (on login)
const token = await new SignJWT({ userId: user.id, role: user.role })
  .setProtectedHeader({ alg: 'HS256' })
  .setExpirationTime('7d')
  .sign(SECRET)

cookies().set('session', token, { httpOnly: true, secure: true, sameSite: 'lax' })

// VERIFY TOKEN (on each request)
async function getSession() {
  const token = cookies().get('session')?.value
  if (!token) return null
  try {
    const { payload } = await jwtVerify(token, SECRET)
    return payload // { userId, role, exp }
  } catch {
    return null // expired or tampered
  }
}

// LOGOUT — just delete the cookie
// ⚠️ The token is still cryptographically valid until it expires
cookies().delete('session')
```

## JWT Gotcha
The **main gotcha with JWT** is logout — deleting the cookie stops the browser sending it, but the token itself is still technically valid until it expires. For most personal projects that's fine, but worth knowing.

Why It Happens in Your Code

Look closely at your middleware logic:

```typescript
// inside middleware.ts
const session = await getSession() 
// This ONLY decodes the cookie and checks the crypto signature.
// It never talks to your database!
```
If you ever need "log out all devices" or "ban this user immediately" — you'd need a DB blocklist anyway, which defeats the stateless benefit.

that's the key advantage of DB sessions.

When you log out with DB sessions:
```js
// delete the session row from DB
await db.session.delete({ where: { token } })
```

That token is **instantly dead.** Even if someone stole it, it's useless the moment you delete it from the DB.

---

## The JWT revocation problem

Because the server stores nothing, a JWT can't be invalidated mid-life. Three workarounds, each with a cost:

| Strategy | How it works | Cost |
|----------|-------------|------|
| Short expiry | Set `exp` to 5–15 min | User re-authenticates constantly |
| Redis blocklist | Store revoked JTIs in Redis, check each request | ~2ms extra lookup — but now stateful |
| Session version | Store `sessionVersion` in DB + JWT, reject on mismatch | DB hit on sensitive routes only |

> Both Redis blocklist and session version reintroduce a server-side store — which defeats the main point of JWT. If you need any of these, a plain DB session is simpler.

# DB session vs JWT

|  | DB session | JWT |
|--|-----------|-----|
| Install | none | `jose` |
| Instant revocation | ✅ yes | ❌ hard |
| DB hit per request | 1 indexed query | none |
| Multiple servers | needs shared DB | works out of the box |
| Logout | delete row → token dead instantly | delete cookie → token valid until expiry |
| Ban user / all devices | trivial | needs a blocklist DB |

> JWT's stateless benefit only matters at massive scale. For personal projects, that one DB lookup per request is negligible — under 1ms on an indexed column. **DB sessions are the safer, simpler default.**

## Real world scenarios where DB sessions win:

- **User changes password** → delete all their sessions → all devices logged out immediately
- **You ban a user** → delete their sessions → they're out right now
- **"Log out all devices"** → trivial, just delete all rows for that userId
- **Token gets stolen** → you can invalidate it instantly

## So the real tradeoff is:

| | DB Session | JWT |
|---|---|---|
| Instant invalidation | ✅ | ❌ |
| DB hit per request | yes | no |

And for a personal project, that **one DB lookup per request is nothing.** It's a single indexed query, takes under a millisecond.

JWT's stateless benefit only really matters at **massive scale** with many servers.
---

## Hybrid Pattern — Cookie-Based JWT (Gold Standard for Next.js)

Store a JWT **inside** an `httpOnly` cookie. This combines the best of both worlds:

| Feature | Session + Cookie | Pure JWT (localStorage) | Cookie-Based JWT |
|---|---|---|---|
| Where is data stored? | Server DB | Token payload | **Token payload** |
| DB lookup per request? | Yes | No | **No** |
| XSS protection? | High | Low | **High** |
| Instant revocation? | Easy | Hard | **Hard (needs Redis)** |
| Mobile friendly? | Harder | Excellent | **Excellent** |

### Why this works well with Next.js middleware

Next.js middleware runs on the **Edge** before rendering any page. A DB lookup there adds latency. JWT verification is instant — no DB query — so routes stay fast.

```ts
// middleware.ts
export async function middleware(req: NextRequest) {
  const session = await getSession() // just decodes + verifies signature

  if (req.nextUrl.pathname.startsWith('/dashboard') && !session) {
    return NextResponse.redirect(new URL('/login', req.url))
  }
}
```
---

## Structured Code — Express → Next.js

### Express: Traditional Session Auth

```javascript
// server.js
import express from 'express'
import session from 'express-session'
import bcrypt from 'bcrypt'

const app = express()

// 1. COOKIE + SESSION SETUP
app.use(session({
  secret: process.env.SESSION_SECRET,   // signs the session cookie
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,    // JS cannot read this cookie (XSS protection)
    secure: true,      // only sent over HTTPS
    sameSite: 'lax',   // CSRF protection
    maxAge: 1000 * 60 * 60 * 24  // 1 day
  }
}))

// 2. LOGIN ROUTE — verify and establish session
app.post('/login', async (req, res) => {
  const { email, password } = req.body
  const user = await db.findUser(email)

  if (!user || !await bcrypt.compare(password, user.hashedPassword)) {
    return res.status(401).json({ error: 'Invalid credentials' })
  }

  req.session.userId = user.id   // ← this is what gets stored server-side
  res.json({ ok: true })
})

// 3. PROTECT ROUTES — middleware
function requireAuth(req, res, next) {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Not authenticated' })
  }
  next()
}

app.get('/profile', requireAuth, async (req, res) => {
  const user = await db.findById(req.session.userId)
  res.json(user)
})

// 4. LOGOUT — destroy session instantly
app.post('/logout', (req, res) => {
  req.session.destroy()
  res.clearCookie('connect.sid')
  res.json({ ok: true })
})
```

---

### Express: JWT (Stateless Auth)

```javascript
import jwt from 'jsonwebtoken'
const SECRET = process.env.JWT_SECRET

// LOGIN — create and send JWT
app.post('/login', async (req, res) => {
  const user = await verifyCredentials(req.body)
  if (!user) return res.status(401).json({ error: 'Invalid' })

  const token = jwt.sign(
    { userId: user.id, role: user.role },  // payload — never put passwords here
    SECRET,
    { expiresIn: '1d' }
  )

  // Option A: httpOnly cookie (recommended)
  res.cookie('token', token, { httpOnly: true, secure: true, sameSite: 'lax' })

  // Option B: response body (client stores in memory/localStorage)
  res.json({ token })
})

// MIDDLEWARE — verify JWT on protected routes
function requireAuth(req, res, next) {
  // reads from cookie OR Authorization header
  const token = req.cookies.token || req.headers.authorization?.split(' ')[1]
  if (!token) return res.status(401).json({ error: 'No token' })

  try {
    const decoded = jwt.verify(token, SECRET)  // throws if invalid or expired
    req.user = decoded   // { userId, role, iat, exp }
    next()
  } catch {
    res.status(401).json({ error: 'Invalid token' })
  }
}

app.get('/profile', requireAuth, async (req, res) => {
  const user = await db.findById(req.user.userId)  // identity comes from token
  res.json(user)
})
```
---

### Next.js App Router — Complete Pattern

```
app/
├── api/auth/
│   ├── login/route.ts
│   └── logout/route.ts
├── lib/
│   ├── auth.ts        ← core auth logic
│   └── session.ts     ← session helpers
├── middleware.ts       ← route protection (runs on every request)
└── (protected)/
    └── dashboard/page.tsx
```

```typescript
// lib/session.ts — JWT stored in httpOnly cookie
import { SignJWT, jwtVerify } from 'jose'
import { cookies } from 'next/headers'

const secret = new TextEncoder().encode(process.env.JWT_SECRET)

export async function createSession(userId: string) {
  const token = await new SignJWT({ userId })
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('1d')
    .sign(secret)

  cookies().set('session', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 60 * 60 * 24
  })
}

export async function getSession() {
  const token = cookies().get('session')?.value
  if (!token) return null

  try {
    const { payload } = await jwtVerify(token, secret)
    return payload  // { userId, exp }
  } catch {
    return null  // expired or tampered
  }
}

export async function deleteSession() {
  cookies().delete('session')
}
```

```typescript
// api/auth/login/route.ts
import { NextResponse } from 'next/server'
import { createSession } from '@/lib/session'

export async function POST(req: Request) {
  const { email, password } = await req.json()
  const user = await verifyUser(email, password)

  if (!user) {
    return NextResponse.json({ error: 'Invalid credentials' }, { status: 401 })
  }

  await createSession(user.id)  // sets the httpOnly cookie internally
  return NextResponse.json({ ok: true })
}
```

```typescript
// middleware.ts — runs on EVERY request before rendering
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
import { getSession } from '@/lib/session'

export async function middleware(req: NextRequest) {
  const session = await getSession()  // decodes + verifies JWT, no DB query

  const isProtected = req.nextUrl.pathname.startsWith('/dashboard')
  const isAuthPage  = req.nextUrl.pathname.startsWith('/login')

  if (isProtected && !session) {
    return NextResponse.redirect(new URL('/login', req.url))
  }

  if (isAuthPage && session) {
    return NextResponse.redirect(new URL('/dashboard', req.url))
  }

  return NextResponse.next()
}

export const config = {
  matcher: ['/dashboard/:path*', '/login']
}
```

```typescript
// dashboard/page.tsx — Server Component, reads session directly
import { getSession } from '@/lib/session'
import { redirect } from 'next/navigation'

export default async function DashboardPage() {
  const session = await getSession()
  if (!session) redirect('/login')  // second safety layer beyond middleware

  const user = await db.findById(session.userId as string)
  return <div>Welcome {user.name}</div>
}
```

## When to use what

| Situation | Use |
|---|---|
| Personal project / learning | DB Sessions (simplest, safest) |
| SPA or mobile app calling an API | JWT (stateless, header-based) |
| Next.js full-stack app | Cookie-Based JWT (hybrid) |
| Multiple servers / microservices | JWT (stateless scales better) |
| Need instant revocation | DB Sessions or JWT + Redis blacklist |

---

## Don't Roll Your Own Auth in Production

Auth has many subtle failure modes:
- Password hashing cost factors (bcrypt, argon2)
- Timing-safe comparisons to prevent enumeration
- CSRF, session fixation, brute force protection
- OAuth flows are genuinely complex

**Use a library instead:**

| Situation | Recommended |
|---|---|
| Side project / startup | **Clerk** (best DX) or **Auth.js** |
| Already using Supabase | **Supabase Auth** |
| Full control, self-hosted | **Auth.js** with DB adapter |
| Enterprise / SSO | **Auth0** or **WorkOS** |

> Auth is not a feature you build — it is a dependency you configure.

---

## Full Hierarchy

```
Custom (you write everything)
  ↓ more abstraction
Library — Auth.js, Better Auth (your server, your DB)
  ↓ more abstraction
Hosted Service — Clerk, Auth0 (their servers, their DB)
```
---