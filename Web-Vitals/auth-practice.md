## Auth Practice
Good way to learn. Here's the fundamentals broken down simply:
---
So your only installs for auth are:

```bash
npm install bcrypt
npm install prisma @prisma/client  # if not already using it
```
---

## The Pieces You Need

### 1. Password Hashing
Never store plain passwords. Use `bcrypt`:
```js
import bcrypt from 'bcrypt'

// on register
const hash = await bcrypt.hash(password, 12) // 12 = cost factor
// save hash to DB, never the raw password

// on login
const match = await bcrypt.compare(inputPassword, hashFromDB)
if (!match) return 'wrong password'
```

### 2. Session Management
After login, you need to remember who the user is. Two approaches:

**Option A — DB Session (simpler, recommended for beginners)**
```
login → generate random token → save token in DB → send token in cookie
each request → read cookie → look up token in DB → get user
logout → delete token from DB
```

**Option B — JWT (stateless)**
```
login → sign a JWT with user id → send in cookie
each request → verify JWT signature → decode user id
logout → (token lives until expiry, handle client side)
```

For a personal project, **DB sessions are easier to reason about.**

### 3. The Cookie
```js
// set it like this (in Next.js API route or server action)
res.setHeader('Set-Cookie', [
  `session=${token}; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=604800`
])
```

- `HttpOnly` — JS can't read it (blocks XSS)
- `Secure` — only sent over HTTPS
- `SameSite=Strict` — blocks CSRF

**Store the token in a cookie, not localStorage.**

---

## Minimal Example Stack (Next.js)

```
DB:       Prisma + SQLite (or Postgres)
Hashing:  bcrypt
Sessions: store in DB, token in cookie
```

Schema:
```prisma
model User {
  id       String    @id @default(cuid())
  email    String    @unique
  password String    // hashed
  sessions Session[]
}

model Session {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  expiresAt DateTime
}
```

Register route:
```js
const existing = await db.user.findUnique({ where: { email } })
if (existing) return error('Email taken')

const hash = await bcrypt.hash(password, 12)
const user = await db.user.create({ data: { email, password: hash } })
```

Login route:
```js
const user = await db.user.findUnique({ where: { email } })
if (!user) return error('Invalid credentials') // same message for security

const match = await bcrypt.compare(password, user.password)
if (!match) return error('Invalid credentials')

const token = crypto.randomBytes(32).toString('hex')
await db.session.create({
  data: { userId: user.id, token, expiresAt: /* 7 days from now */ }
})

// set token in httpOnly cookie
```

Auth check (middleware or server component):
```js
const token = cookies().get('session')?.value
if (!token) redirect('/login')

const session = await db.session.findUnique({ where: { token } })
if (!session || session.expiresAt < new Date()) redirect('/login')

// session.userId is your logged in user
```

---

Here's the JWT approach from scratch:

## How JWT Works

```
login → create token with user data baked in → send in cookie
request → verify token signature → decode user data directly
no DB lookup needed for auth ✓
```

---

## Install

```bash
npm install jose    # recommended, works in Next.js edge runtime
# OR
npm install jsonwebtoken  # older, Node.js only
```

`jose` is preferred for Next.js because `jsonwebtoken` doesn't work in edge middleware.

---

## The Flow in Code

### 1. Create a secret key
```js
// lib/auth.js
const SECRET = new TextEncoder().encode(process.env.JWT_SECRET)
// in .env → JWT_SECRET=some_long_random_string_here
```

### 2. Register (same as before, no change)
```js
const hash = await bcrypt.hash(password, 12)
await db.user.create({ data: { email, password: hash } })
```

### 3. Login → create and sign the token
```js
import { SignJWT } from 'jose'

const token = await new SignJWT({ userId: user.id, email: user.email })
  .setProtectedHeader({ alg: 'HS256' })
  .setExpirationTime('7d')
  .sign(SECRET)

// set in cookie
cookies().set('session', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 60 * 60 * 24 * 7  // 7 days
})
```

### 4. Verify on each request
```js
import { jwtVerify } from 'jose'

async function getUser() {
  const token = cookies().get('session')?.value
  if (!token) return null

  try {
    const { payload } = await jwtVerify(token, SECRET)
    return payload  // { userId, email, exp }
  } catch (err) {
    return null  // invalid or expired token
  }
}
```

### 5. Protect a route
```js
// in a server component or API route
const user = await getUser()
if (!user) redirect('/login')

// use user.userId from here
```

### 6. Logout
```js
// just delete the cookie — no DB needed
cookies().delete('session')
```
---

## What You'll Learn From This

- How hashing works and why it matters
- Cookie security attributes
- DB design for auth
- Protecting routes

Once you've built this once, you'll understand exactly what Auth.js or Clerk are doing under the hood — and you'll appreciate why you'd use them for real projects.