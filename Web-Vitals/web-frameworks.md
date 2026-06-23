# 🌐 Web Frameworks

> **Core idea:** A web framework is structured tooling that handles the repeating parts of building for the web — routing, rendering, data, auth — so you write the parts that are actually unique to your product.

---

## Where It Started — Static Files

The earliest websites were just files in a folder on a machine:

```
www/
├── index.html
├── about.html
├── css/
│   └── style.css
├── js/
│   └── main.js
└── images/
    └── logo.png
```

You can open `index.html` by double-clicking it and it renders in your browser — but that's not hosting. Your browser is just reading a file off your own disk, the same way it would open a PDF. The URL looks like `file:///Users/you/www/index.html`. Nobody else can reach it.

To make it accessible over a network, something needs to listen on a **port** — a numbered channel your OS uses to route incoming connections. A web server does exactly this: it binds to port 80 (HTTP) or 443 (HTTPS) and waits for requests. When one arrives, it maps the URL to a file and sends it back.

To abstract all the configuration to your system to host your site on the network Apache or Nginx web server softwares can locally host your files and become reachable at `http://localhost`. But localhost only accepts connections from the same machine. The rest of the world still can't reach you.

To get onto the public internet you need:
- A machine with a public IP address — so requests can find it
- A web server running on that machine — so it can respond to them
- A domain name pointed at that IP — so people don't type numbers

That's what a hosting provider gives you. Platforms like **GitHub Pages**, **Netlify**, and **Cloudflare Pages** handle all three — you push your files, they take care of the machine, the web server, and the routing. Nginx (or equivalent) is still running at the edge, you just never touch it.

---

## The Client Gets Dynamic — JavaScript Arrives

Static files were just that — static. The same file went to every visitor. When **JavaScript arrived in browsers in 1995**, that changed on the client side.

With JS, a static site could now:
- Validate a form before submitting it
- Show and hide elements without reloading the page
- Animate things and respond to clicks
- Run calculations in the browser

But there was a hard ceiling. JavaScript ran in the browser and had no way to reach anything outside it. It couldn't:
- Talk to a database
- Remember who you were between visits
- Check a password against stored credentials
- Process a payment or write anything to disk
- Personalise a page — every visitor still got the same file

The browser was a sealed box. Anything that needed persistence, security, or data had to happen somewhere else.

---

## The Server Gets Dynamic — PHP and Apache

PHP opened that box from the other side. It embedded directly into Apache — when a request came in for `profile.php`, Apache would execute the file on the server and send back the generated HTML:

```
Browser → Apache → runs profile.php → queries database → HTML generated → sent back
```

The PHP file could read from a database, check a session, look up the logged-in user, and build a personalised page on the fly. The browser just displayed the result — it had no idea any of this happened.

Ruby on Rails pushed this further — instead of logic scattered across PHP files, it gave you a structured pattern: models handle data, controllers handle requests, views handle HTML. This was the beginning of web server frameworks as organised systems rather than scripts.

---

### Client-Side Frameworks

A client-side framework is code that runs in the browser and handles how the UI behaves, updates, and responds to the user — without a full page reload from the server. They split into two distinct camps.

**Enhancement frameworks** — add behaviour to server-rendered HTML

| Framework | What It Does |
|---|---|
| **htmx** | HTML attributes trigger server requests and swap in the response. The server still generates HTML; htmx is the JS library doing the swapping. |
| **Stimulus** | Small JS controllers that attach behaviour to server-rendered HTML elements. |
| **Alpine.js** | Sprinkles reactivity and state directly in HTML attributes. No build step, no virtual DOM. |

**SPA frameworks** — generate all HTML in the browser from JavaScript

| Framework | What It Does |
|---|---|
| **Svelte** | Compiles components to vanilla JS at build time. No runtime shipped to the browser. |
| **Vue** | Component-based UI rendered entirely in the browser. |
| **React** | Full component tree rendered in the browser. Server sends one empty HTML file, React builds everything client-side. |
| **Angular** | Opinionated full SPA framework with its own module system, dependency injection, and tooling. |

---

### Minimal / Enhancement Frameworks

Assumes HTML already exists on the page — delivered by a web server — and adds behaviour to it. Nothing is generated in the browser.

Example: **Alpine.js**

```
Server sends:
└── Complete HTML page

Alpine adds:
├── Reactive state       (x-data)
├── Event handling       (x-on:click)
├── Show/hide elements   (x-show)
└── DOM updates          (x-text)
```

No build step. Drop a script tag in and it works. The server is still doing the heavy lifting — Alpine just handles the interactive layer on top.

---

### SPA Frameworks

The server sends one nearly empty HTML file. The framework generates all the HTML itself in the browser using JavaScript.

Example: **React**

```
Server sends:
└── <div id="root"></div>  ← empty shell

React builds:
├── Component tree
├── Routing (React Router)
├── State management
└── All HTML the user sees
```

The entire UI is JavaScript. If JS is disabled or slow to load, the user sees nothing. The tradeoff for that cost is a fully interactive, app-like experience with no page reloads.

---

## Meta Frameworks — The Line Blurs Again

Before going further: **SSR** (server-side rendering) means the server generates the HTML for each request. **SSG** (static site generation) means HTML is generated once at build time. **CSR** (client-side rendering) is what SPAs do — the browser generates it. SPAs are pure CSR, which is why SEO tools and slow connections struggle with them.

Meta frameworks like **Next.js** (React) and **Remix** (React) or **SvelteKit** (Svelte) solve this by running code on *both* sides — the same framework handles server rendering, API routes, and client interactivity. A page can be rendered on the server for the first load, then the client takes over for navigation.

```
Next.js
├── Server-side rendering   → fast initial load, good SEO
├── API routes              → your backend endpoints
├── Client components       → interactive UI in the browser
└── Static generation       → pre-build pages at deploy time
```

The server/client split is no longer a hard line — it's a decision you make per page or per component.

---

## Batteries-Included Frameworks — The Full Stack in One

Meta frameworks blur the server/client line. Batteries-included frameworks go further: they give you the entire stack — database access, auth, API, and UI — as one cohesive system with conventions for how everything fits together.

The idea is that most apps need the same things: a database, user accounts, forms, data fetching, and some UI. Rather than assembling these yourself from separate packages, a batteries-included framework makes all of those decisions for you and wires them together.

**Redwood.js** is the clearest example:

```
Redwood
├── React frontend (cells for data fetching)
├── GraphQL API (auto-generated from your schema)
├── Prisma ORM (database access)
├── Auth (pluggable providers)
├── Storybook (component development)
└── Jest (testing)
```

One `yarn create redwood-app` gets you all of that, with an opinionated file structure and generators for common patterns.

Other examples: **Blitz.js** (adds direct database access to Next.js, no API layer needed), **Wasp** (a config-driven full stack framework for React + Node).

---

## Web Server Frameworks

A web server framework is structured tooling built on top of whatever HTTP solution the runtime provides.

| Raw HTTP Layer | Framework Layer | Your Code |
|---|---|---|
| Node `http` module | Express / Hono | Route handlers |
| Ruby (standard interface) | Rails / Sinatra | Controllers |
| Python (standard interface) | Django / Flask | Views |
| Go `net/http` | Gin / Echo | Handlers |
| PHP SAPI | Laravel / Symfony | Controllers |
| Java Servlet / Netty | Spring / Quarkus | Controllers |
| Elixir Cowboy | Phoenix | Controllers |
| .NET Kestrel | ASP.NET Core / MVC | Controllers |

Every framework provides the same core things: a **router** (map URLs to functions), a **middleware pipeline** (run code before/after your handler), a **request/response abstraction** (structured objects instead of raw bytes), and conventions so teams don't reinvent the wheel.

> All of these frameworks are capable of sending HTML back to the browser — that's what they were originally built to do. Today most teams use them as a pure API layer, sending JSON instead, with a separate client framework handling the UI.

### Minimal / Routing Frameworks

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

### Full Web Server Frameworks

Comes with an ORM, migration system, authentication, authorization, background jobs, email, file storage, and caching — all pre-integrated and opinionated about how they fit together.

Examples: Laravel (PHP), Django (Python), Ruby on Rails, NestJS (Node.js)

---

### The Split — Client Frameworks Take Over Rendering

As JavaScript matured, the browser became capable enough to handle rendering entirely on its own. Instead of the server generating HTML, the server sends JSON data and the client builds the UI from it:

```
PHP/Rails era:   Server → HTML → Browser displays it
Modern era:      Server → JSON → Client framework builds the UI
```

This shift created an entire category of frameworks that run only in the browser.

---

## The Full Landscape

```
Minimal client      Alpine.js
                        ↓
Component / SPA     React, Vue, Svelte, Angular
                        ↓
Meta frameworks     Next.js, Remix, SvelteKit
                        ↓
Batteries included  Redwood, Blitz, Wasp
```

Each step down trades flexibility for convention. Each step up trades convention for control.