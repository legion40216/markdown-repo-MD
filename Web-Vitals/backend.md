# Backend Development

## What Is a Backend?

A backend is the **safety and logic layer** that sits between users and your database. Users never interact with your data directly — every request passes through the backend first, where it gets verified, processed, and controlled before anything is read or written.

Think of it this way: your database is a vault, your frontend is the lobby, and the backend is everything in between — the security desk, the ID check, the rules engine, and the paper trail.

---

## Core Responsibilities

At its heart, every backend must answer a few fundamental questions on every request:

- **Authentication** — *Who are you?* Verify the identity of the caller.
- **Authorization** — *What are you allowed to do?* Enforce roles and permissions.
- **Input Validation** — *Is this data safe and well-formed?* Reject bad or malicious input before it reaches the database.
- **Business Logic** — *What rules apply?* Pricing, calculations, workflows, and constraints all live here.
- **Rate Limiting** — *Is this caller abusing the system?* Throttle excessive requests to prevent abuse.
- **Secret Management** — *Are credentials protected?* Database passwords, API keys, and tokens must never be exposed to the client.

---

## What a Backend Actually Does

### 1. Routing

The backend maps incoming URLs to the code that should handle them. Every endpoint is a contract between the client and the server.

```
GET  /users          → fetch all users
GET  /products       → fetch all products
POST /products       → create a new product
GET  /orders/123     → fetch a specific order
```

HTTP methods (GET, POST, PUT, PATCH, DELETE) define the *intent* of the request, while the URL path defines the *resource* being acted on.

---

### 2. Request & Response Handling

The backend parses everything the client sends and constructs a structured reply. Incoming data can arrive as:

- **JSON** — the dominant format for APIs
- **Form data** — from HTML forms
- **Cookies and headers** — for auth tokens, metadata, preferences
- **File uploads** — images, documents, videos

The response is then built and sent back — typically JSON for APIs, or fully rendered HTML in traditional architectures.

---

### 3. Database Management & ORMs

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

### 4. Authentication

Authentication is the process of verifying who a user is. Modern backends support multiple strategies:

- **Email + Password** — the classic approach, using hashed and salted passwords
- **Sessions** — the server stores login state; a cookie references it
- **JWT (JSON Web Tokens)** — a stateless, self-contained token the client holds and sends on every request
- **OAuth** — delegated authentication via providers like Google, GitHub, or Facebook

Each strategy has trade-offs between security, scalability, and complexity.

---

### 5. Authorization

Authentication tells you *who* someone is. Authorization tells you *what they're allowed to do*. Most systems implement **Role-Based Access Control (RBAC)**:

| Role | Permissions |
|---|---|
| Admin | Full access — read, write, delete, manage users |
| Moderator | Can manage content, cannot manage users or billing |
| Customer | Can only access their own data |

Authorization logic runs after authentication and before any business logic executes.

---

### 6. Validation

Before any data reaches the database, it must be checked. Validation ensures:

- Required fields are present
- Email addresses are properly formatted
- Passwords meet minimum length requirements
- Numerical values fall within expected ranges
- No malicious payloads are embedded in the input

Skipping validation is one of the most common sources of bugs and security vulnerabilities.

---

### 7. File Storage Systems

Backends handle file uploads — profile pictures, documents, videos — and decide where to store them. Full-featured backends provide a **storage abstraction layer**, letting you switch between:

- **Local disk** — simple, useful for development
- **Cloud storage** — AWS S3, Google Cloud Storage, or Azure Blob Storage for production scale

The abstraction means changing your storage provider is a single config change, not a rewrite.

---

### 8. Background Jobs & Queues

Some operations are too slow to block a user on. If someone uploads a video or requests a large report, you can't make them stare at a spinner for 30 seconds. Background job queues solve this:

- Email delivery
- PDF generation
- Image resizing and optimization
- AI processing tasks
- Importing large datasets

The request returns immediately with a "processing" status, and the heavy work happens asynchronously in the background.

---

### 9. Caching

Repeatedly querying the database for the same data is wasteful. Caching stores the result of expensive operations so future requests can be served almost instantly.

```
Request → Cache hit? → Return instantly
        → Cache miss? → Query database → Store in cache → Return
```

Common caching tools include **Redis** and **Memcached**. Caching is especially powerful for read-heavy data like product listings, leaderboards, or configuration settings.

---

### 10. Session Management

HTTP is stateless — every request is, by default, independent. Session management is how backends give the illusion of continuity between requests. Sessions track:

- Who is logged in
- Shopping cart contents
- User preferences
- Multi-step form progress

Sessions can be stored server-side (in a database or cache) or encoded in a stateless JWT the client holds.

---

### 11. Security

The backend is the primary defense layer against attacks. Common protections include:

| Threat | Defense |
|---|---|
| **SQL Injection** | Parameterized queries, ORMs |
| **XSS (Cross-Site Scripting)** | Output encoding, Content Security Policy |
| **CSRF (Cross-Site Request Forgery)** | CSRF tokens, SameSite cookies |
| **Brute Force** | Rate limiting, account lockouts |
| **Data Exposure** | Encrypted connections (HTTPS/TLS), field-level access control |

Security is not a single feature — it's a discipline applied across every layer of the backend.

---

### 12. Email Systems

Backends integrate with email delivery services to send:

- Password reset links
- Email verification on sign-up
- Transactional notifications (order confirmations, invoices)
- Alerts and system notifications

These are almost always handled asynchronously via background jobs, since email delivery can be slow or unreliable.

---

### 13. Scheduled Tasks (Cron Jobs)

Some work isn't triggered by a user — it runs on a schedule. Backends use cron jobs to automate recurring operations:

- **Hourly** — sync data with external APIs
- **Daily** — generate usage reports, send digest emails
- **Monthly** — generate invoices, archive old records
- **On demand** — clean up expired sessions, purge soft-deleted records

---

### 14. Real-Time Communication

Traditional request-response HTTP isn't enough for live features. **WebSockets** maintain a persistent, two-way connection between client and server, enabling:

- Chat and messaging
- Live notifications
- Real-time dashboards and analytics
- Multiplayer features
- Collaborative editing

---

### 15. Code Organization

As backends grow, structure becomes critical. Backends typically organize logic into distinct layers:

```
controllers/    → Handle HTTP requests and responses
services/       → Business logic and rules
repositories/   → Database access and queries
```

Patterns like **Dependency Injection** help keep these layers decoupled and testable, so changes in one layer don't break the others.

---

## Two Kinds of Backend Frameworks

Not all backend frameworks are the same. They fall on a spectrum from lean and minimal to fully loaded.

---

### Minimal / API Frameworks

A minimal framework is like a **post office** — it takes a request, routes it to the right handler, and sends a response. It's fast, unopinionated, and gives you a clean slate. You wire in everything else yourself.

**Examples:** Express.js, FastAPI, Flask, Hono.js

A typical Express.js setup might look like:

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

Minimal frameworks give you maximum flexibility, but require more decisions and assembly work upfront.

---

### Full Backend Frameworks

A full framework is the **entire city infrastructure** behind the post office. It comes with an ORM, migration system, authentication, authorization, background jobs, email, file storage, caching, and a security layer — all pre-integrated and opinionated about how they work together.

**Examples:** Laravel (PHP), Django (Python), Ruby on Rails, NestJS (Node.js)

These frameworks follow the **MVC (Model-View-Controller)** pattern, traditionally capable of both serving API data *and* rendering full HTML pages server-side using template engines.

---

### Comparison

| | Minimal Framework | Full Framework |
|---|---|---|
| **Setup** | Fast, bare-bones | More initial config, more included |
| **Flexibility** | High — you choose every tool | Lower — conventions are set |
| **Built-in ORM** | No | Yes |
| **Built-in Auth** | No | Yes |
| **Background Jobs** | No | Yes |
| **File Storage** | No | Yes |
| **Best for** | APIs, microservices, custom stacks | Monoliths, rapid full-stack development |

---

## The Shift Toward API-Only Backends

Historically, backends did everything — including rendering the HTML pages users saw. Today, that's changed.

Modern development has decoupled the frontend from the backend. A frontend framework like **React** or **Vue** handles all UI rendering in the browser, and the backend becomes a pure **API provider** — sending JSON, not HTML.

This means even teams using powerful full frameworks like Django or Laravel may only use them as API backends, with a separate React frontend consuming the data.

> Even when a backend *only* serves JSON, it is still handling authentication, authorization, database management, background jobs, security, and caching behind the scenes. The API surface is just the visible tip of a large iceberg.

---

## Summary

The backend is the nervous system of any application. Whether you choose a minimal framework and build your stack piece by piece, or reach for a full framework that gives you everything out of the box, the responsibilities are the same: keep data safe, enforce rules, handle complexity, and make the frontend's job simple.