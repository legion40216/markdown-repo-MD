# Backend Development

## What Is a Backend?

A backend is the **safety and logic layer** that sits between users and your database. Users never interact with your data directly — every request passes through the backend first, where it gets verified, processed, and controlled before anything is read or written.

Think of it this way: your database is a vault, your frontend is the lobby, and the backend is everything in between — the security desk, the ID check, the rules engine, and the paper trail.

At this point in the request lifecycle, the web server has already handled the socket connection, parsed raw bytes into a structured HTTP request, and terminated SSL. What arrives at your code is a clean object: a method, a path, headers, and a body. Everything from here is yours.

---

## What a Backend Actually Does

### 1. Routing

The backend maps incoming URLs to the code that should handle them.

```
GET  /users          → fetch all users
GET  /products       → fetch all products
POST /products       → create a new product
GET  /orders/123     → fetch a specific order
```

HTTP methods (GET, POST, PUT, PATCH, DELETE) define the *intent* of the request, while the URL path defines the *resource* being acted on.

---

### 2. Authentication

Authentication is the process of verifying who a user is. Modern backends support multiple strategies:

- **Email + Password** — the classic approach, using hashed and salted passwords
- **Sessions** — the server stores login state; a cookie references it
- **JWT (JSON Web Tokens)** — a stateless, self-contained token the client holds and sends on every request
- **OAuth** — delegated authentication via providers like Google, GitHub, or Facebook

Each strategy has trade-offs between security, scalability, and complexity.

---

### 3. Authorization

Authentication tells you *who* someone is. Authorization tells you *what they're allowed to do*. Most systems implement **Role-Based Access Control (RBAC)**:

| Role | Permissions |
|---|---|
| Admin | Full access — read, write, delete, manage users |
| Moderator | Can manage content, cannot manage users or billing |
| Customer | Can only access their own data |

Authorization runs after authentication and before any business logic executes.

---

### 4. Validation

Before any data reaches the database, it must be checked. Validation ensures:

- Required fields are present
- Email addresses are properly formatted
- Passwords meet minimum length requirements
- Numerical values fall within expected ranges
- No malicious payloads are embedded in the input

Skipping validation is one of the most common sources of bugs and security vulnerabilities.

---

### 5. Database Management & ORMs

The backend is the sole gateway to your database. Rather than writing raw SQL everywhere, most backends use an **ORM (Object-Relational Mapper)** — a tool that lets you interact with your database using your programming language's native syntax.

```sql
-- Raw SQL
SELECT * FROM users WHERE id = 1;
```

```javascript
// ORM equivalent
User.findOne({ id: 1 });
```

ORMs also handle **migrations** — versioned, trackable changes to your database schema over time — so you never lose track of how your data structure evolved.

---

### 6. Caching

Repeatedly querying the database for the same data is wasteful. Caching stores the result of expensive operations so future requests can be served almost instantly.

```
Request → Cache hit? → Return instantly
        → Cache miss? → Query database → Store in cache → Return
```

Common caching tools include **Redis** and **Memcached**. Caching is especially powerful for read-heavy data like product listings, leaderboards, or configuration settings.

---

### 7. Session Management

HTTP is stateless — every request is independent by default. Session management is how backends give the illusion of continuity between requests. Sessions track:

- Who is logged in
- Shopping cart contents
- User preferences
- Multi-step form progress

Sessions can be stored server-side (in a database or cache) or encoded in a stateless JWT the client holds — the authentication section covers both approaches.

---

### 8. File Storage

Backends handle file uploads — profile pictures, documents, videos — and decide where to store them. Full-featured backends provide a **storage abstraction layer**, letting you switch between:

- **Local disk** — simple, useful for development
- **Cloud storage** — AWS S3, Google Cloud Storage, or Azure Blob Storage for production scale

The abstraction means changing your storage provider is a single config change, not a rewrite.

---

### 9. Background Jobs & Scheduled Tasks

Some operations are too slow to block a user on. Background job queues solve this by returning immediately with a "processing" status while heavy work happens asynchronously:

- Email delivery
- PDF generation
- Image resizing
- AI processing
- Large data imports

Some work isn't triggered by a user at all — it runs on a fixed schedule using **cron jobs**:

- **Hourly** — sync data with external APIs
- **Daily** — generate usage reports
- **Monthly** — generate invoices, archive old records
- **On demand** — clean up expired sessions, purge soft-deleted records

---

### 10. Real-Time Communication

Traditional request-response HTTP isn't enough for live features. **WebSockets** maintain a persistent, two-way connection between client and server, enabling:

- Chat and messaging
- Live notifications
- Real-time dashboards
- Multiplayer features
- Collaborative editing

---

### 11. Security

The backend is the primary defense layer. Auth and validation handle two major attack surfaces — but several others require explicit protection:

| Threat | Defense |
|---|---|
| **SQL Injection** | Parameterized queries, ORMs |
| **XSS (Cross-Site Scripting)** | Output encoding, Content Security Policy |
| **CSRF (Cross-Site Request Forgery)** | CSRF tokens, SameSite cookies |
| **Brute Force** | Rate limiting, account lockouts |
| **Data Exposure** | HTTPS/TLS, field-level access control |
| **Credential Leaks** | Secret management — DB passwords and API keys must never reach the client |

Security is not a single feature — it's a discipline applied across every layer of the backend.

---

### 12. Code Organization

As backends grow, structure becomes critical. Backends typically organize logic into distinct layers:

```
controllers/    → Handle HTTP requests and responses
services/       → Business logic and rules
repositories/   → Database access and queries
```

Patterns like **Dependency Injection** keep these layers decoupled and testable, so changes in one layer don't break the others.

---

## Two Kinds of Backend Frameworks

Not all backend frameworks are the same. They fall on a spectrum from lean and minimal to fully loaded.

Backend Framework Landscape
Minimal / API Frameworks
Handles routing, middleware, and request/response — nothing else. You wire in every other piece yourself.
Examples: Express.js, FastAPI, Flask, Hono.js
```
Express.js
├── Routing ✓
├── Middleware ✓
└── Request/Response handling ✓

You add:
├── Prisma     → ORM & database access
├── Passport   → Authentication
├── BullMQ     → Background job queues
├── Multer     → File uploads
└── Redis      → Caching
```
Maximum flexibility, but more upfront decisions and assembly work.

Full Backend Frameworks
Comes with an ORM, migration system, authentication, authorization, background jobs, email, file storage, and caching — all pre-integrated and opinionated about how they fit together. These run entirely on the server.
Examples: Laravel (PHP), Django (Python), Ruby on Rails, NestJS (Node.js)

---

## The Shift Toward API-Only Backends

Historically, backends rendered the HTML pages users saw. Today that's changed. A frontend framework like **React** or **Vue** handles all UI rendering in the browser, and the backend becomes a pure **API provider** — sending JSON, not HTML.

This means even teams using powerful full frameworks like Django or Laravel may use them purely as API backends, with a separate React frontend consuming the data.

> Even when a backend *only* serves JSON, it is still handling authentication, authorization, database management, background jobs, security, and caching. The API surface is just the visible tip of a large iceberg.

---

## Summary

The backend is the nervous system of any application. Whether you build your stack piece by piece with a minimal framework, or reach for a full framework that gives you everything out of the box, the job is the same: sit between the user and the data, enforce every rule, and make none of that complexity visible to the frontend.