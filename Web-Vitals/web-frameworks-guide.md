# Web Frameworks — A Developer's Map

> A guide to the specific tools in the web framework landscape: what they're rated, what they're for, and when to reach for each one.

---

## 🟢 Mainstream &nbsp; 🟡 Worth Knowing &nbsp; 🔵 Niche / Specialised

These labels are a guide, not a rule. Mainstream means it's widely used in real-world production and has a strong job market. Niche doesn't mean bad — it means it solves a specific problem exceptionally well.

---

# Part 1 — JavaScript Backend Frameworks

---

## Express.js 🟢 Mainstream

**The grandfather of Node.js frameworks.** Express is minimal by design — it gives you routing and middleware, and nothing else. You build what you need on top of it.

It's been around since 2010, is still the most installed Node.js framework on npm, and underpins dozens of other tools. If you're learning Node.js backend development, Express is where you start.

```js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(3000);
```

That's a working web server. Clean, readable, no ceremony. This simplicity is exactly why it's lasted so long.

**Good for:** REST APIs, SPAs backends, learning server-side JS, rapid prototyping.

**Jump to it when:** You want full control, a huge ecosystem of middleware, and a framework with a decade of Stack Overflow answers behind it.

---

## Fastify 🟢 Mainstream

**Express, but rebuilt for speed.** Fastify uses the same mental model as Express — routes, middleware, handlers — but was engineered from the ground up to be significantly faster. It has built-in schema validation using JSON Schema, which Express doesn't have.

```js
const fastify = require('fastify')();

fastify.get('/', async (request, reply) => {
  return { message: 'Hello World' };
});

fastify.listen({ port: 3000 });
```

The `async/await` approach feels modern, and returning an object automatically serializes it to JSON.

**Good for:** High-performance APIs, microservices, projects where response time matters.

**Jump to it when:** You know Express already and want something faster and more structured without going full enterprise.

---

## NestJS 🟢 Mainstream

**Structure for people who hate chaos.** NestJS is TypeScript-first and heavily inspired by Angular. It uses decorators, dependency injection, and a module system that forces you to organise your code into logical units.

```ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAll();
  }
}
```

The decorator `@Controller('users')` says "this handles `/users` routes." `@Get()` maps to GET requests. The service is injected automatically — you never manually `require` it.

**Good for:** Enterprise apps, SaaS platforms, microservices, real-time apps, teams that need consistency.

**Jump to it when:** You're building something that will grow, needs to be maintained by a team, and needs TypeScript throughout.

---

## Koa.js 🟡 Worth Knowing

**Express, rewritten by the same authors with modern JavaScript.** Koa is ultra-lightweight — it doesn't even ship with a router. Its key feature is how middleware works: code runs on the way *in* and again on the way *out* of a request, which makes things like logging and error handling very clean.

```js
const Koa = require('koa');
const app = new Koa();

app.use(async (ctx, next) => {
  console.log('Before request');
  await next();                  // passes to next middleware
  console.log('After response'); // runs on the way back
});

app.use(async ctx => {
  ctx.body = 'Hello World';
});

app.listen(3000);
```

**Good for:** APIs, microservices, developers who want total control and modern async code.

**Jump to it when:** You find Express's older callback patterns frustrating and want a cleaner minimal core.

---

## Hono 🟡 Worth Knowing

**Built for the edge.** Hono runs on Node.js, Bun, Deno, and Cloudflare Workers — all with the same code. It's tiny, fast, TypeScript-first, and feels like a modern Express.

```ts
import { Hono } from 'hono';
const app = new Hono();

app.get('/', (c) => c.text('Hello World'));

export default app;
```

This same file deploys to Cloudflare Workers globally with no changes.

**Good for:** Edge-deployed APIs, serverless functions, cross-runtime projects.

**Jump to it when:** You're deploying to Cloudflare Workers or want one framework that runs everywhere.

---

## ElysiaJS 🔵 Niche

**Purpose-built for Bun.** ElysiaJS is one of the fastest web frameworks ever benchmarked, but only because it's engineered specifically for the Bun runtime. Its headline feature is end-to-end type safety — your backend types can be shared directly with your frontend.

```ts
import { Elysia } from 'elysia';

const app = new Elysia()
  .get('/', () => 'Hello World')
  .listen(3000);
```

**Good for:** Bun-powered APIs, projects needing maximum performance and type safety across the stack.

**Jump to it when:** You're already using Bun and want bleeding-edge performance.

---

## Hapi.js 🔵 Niche

**Configuration over everything.** Hapi is a framework used at companies like Walmart. It doesn't use external middleware for core features — validation, authentication, and caching are all built in and configured through a rich options object, not middleware chains.

**Good for:** Large enterprise APIs where every feature needs explicit, auditable configuration.

**Jump to it when:** You need a highly configurable, security-focused framework for a big team.

---

## Restify 🔵 Niche

**Express, stripped to the bone, for microservices only.** Restify removes all browser-facing features from Express and focuses purely on HTTP APIs with minimal overhead. Used by Netflix and npm itself.

**Good for:** Internal microservices, API gateways, services where you'll never serve HTML.

**Jump to it when:** You need raw API performance and don't care about anything beyond JSON over HTTP.

---

## Feathers.js 🔵 Niche

**Services, not routes.** Rather than writing routes manually, Feathers lets you define "services" (like a `users` service) and it automatically exposes them as REST and real-time WebSocket endpoints. Great for data-driven apps.

**Good for:** Real-time apps, chat systems, apps that mirror database models as APIs.

---

## Encore.ts 🔵 Niche

**Infrastructure as code — without writing infrastructure.** Encore reads your TypeScript code and automatically provisions databases, queues, and cloud services on AWS or GCP. It's a different paradigm: write your backend logic, and the infra appears.

**Good for:** Cloud-native apps where you want automated infra from your code.

---

# Part 2 — JavaScript Frontend & Meta Frameworks

---

## Next.js 🟢 Mainstream

**The most popular React framework in the world.** Next.js wraps React and adds everything you actually need to ship a production app: server-side rendering, static generation, API routes, image optimisation, and file-based routing.

```jsx
// app/page.jsx — this file IS the homepage route
export default function Home() {
  return <h1>Hello from Next.js</h1>;
}
```

```jsx
// app/api/hello/route.js — this IS an API endpoint
export async function GET() {
  return Response.json({ message: 'Hello' });
}
```

A file in the `app` folder becomes a route automatically. No router config needed.

**Good for:** Production React apps, e-commerce, blogs, dashboards, anything that needs SEO.

**Jump to it when:** You're building a React app that needs to be public, fast, and indexable by search engines.

---

## Nuxt.js 🟢 Mainstream

**Next.js, but for Vue.** Same concept — takes Vue.js and adds file-based routing, SSR, SSG, and a module system. If you prefer Vue's syntax over React's, Nuxt is the production-ready wrapper around it.

**Good for:** Vue-based production apps, content sites, dashboards.

**Jump to it when:** You like Vue's template syntax and want the same batteries-included experience as Next.js.

---

## Astro 🟢 Mainstream

**Ship less JavaScript.** Astro's big idea is "islands architecture" — your pages are mostly static HTML, and only the interactive components load JavaScript. You can mix React, Vue, and Svelte components in the same project.

```astro
---
// This runs on the server at build time
const posts = await fetch('/api/posts').then(r => r.json());
---

<html>
  <body>
    {posts.map(post => <h2>{post.title}</h2>)}
  </body>
</html>
```

The `---` block is server-only. Zero JavaScript is sent to the browser by default.

**Good for:** Blogs, documentation sites, marketing pages, content-heavy sites where performance is critical.

**Jump to it when:** You're building a content site and don't want to ship a full React bundle just to display text.

---

## Ember.js 🟡 Worth Knowing

**Convention over configuration, taken seriously.** Ember was the original "opinionated" JavaScript framework. It has its own data layer (Ember Data), its own router, and a CLI. It's less used than React/Vue but still maintained and popular in certain enterprise and government circles.

**Good for:** Large, complex single-page applications where consistent conventions matter.

**Jump to it when:** You're joining a team already using it, or building a very large SPA with complex data relationships.

---

## Waku 🔵 Niche

**A minimal React Server Components framework.** Waku is an experimental, lightweight framework exploring React's newest primitives. It's not for production yet — it's for developers who want to understand where React is heading.

**Good for:** Learning React Server Components, experimental projects.

---

# Part 3 — Batteries-Included JavaScript Frameworks

---

## AdonisJS 🟡 Worth Knowing

**Laravel, but in JavaScript.** AdonisJS is the closest the JavaScript world has to a "full Laravel experience." It ships with an ORM (Lucid), authentication, validation, mailers, and a templating engine — all first-party.

```ts
Route.post('/login', async ({ request, auth, response }) => {
  const { email, password } = request.only(['email', 'password']);
  await auth.attempt(email, password);
  return response.redirect('/dashboard');
});
```

**Good for:** Full-stack web apps, developers coming from a PHP/Laravel background who want to stay in TypeScript.

**Jump to it when:** You want the "everything works together" feeling of Laravel without leaving JavaScript.

---

## Sails.js 🟡 Worth Knowing

**Rails for Node.js.** Sails follows the MVC pattern, ships with a built-in ORM called Waterline that works across databases, and has WebSocket support out of the box. It can auto-generate REST APIs from your data models.

**Good for:** Real-time apps, data-driven apps, teams familiar with Rails or Django patterns.

---

## Meteor.js 🔵 Niche

**Real-time, full-stack, one language.** Meteor runs the same JavaScript on the client and server and syncs data in real-time automatically. It was revolutionary in 2012, but its all-in-one nature has made it harder to adopt incrementally.

**Good for:** Real-time collaborative apps, prototypes, apps where data sync across clients is the core feature.

**Jump to it when:** You're building a live, multi-user app and want real-time as a first-class feature.

---

## Redwood.js 🔵 Niche

**Full-stack React with GraphQL baked in.** Redwood connects React on the frontend to GraphQL and Prisma on the backend. It has a strong opinion about the whole stack and generates a lot of code for you.

**Good for:** Startups wanting a fast, opinionated full-stack setup with GraphQL.

---

## LoopBack 4 / Total.js 🔵 Niche

Both are enterprise-focused full-stack frameworks for Node.js. LoopBack (from IBM) is built around auto-generating REST and GraphQL APIs from models. Total.js is a Slovak-built framework with its own CMS, NoSQL database, and admin UI. Used in enterprise and government projects in Europe.

**Good for:** Enterprise-grade APIs, content management, specific business application requirements.

---

# Part 4 — Non-JavaScript Frameworks

---

## Django (Python) 🟢 Mainstream

Django is Python's most mature web framework. It ships with an ORM, an admin panel, authentication, form handling, and security protections — all built in and working together. Its philosophy: "don't repeat yourself."

```python
class Post(models.Model):
    title = models.CharField(max_length=200)
    body = models.TextField()

# Django generates the database table,
# the admin UI, and the form validation automatically.
```

**Why it's still relevant:** Django runs Instagram, Pinterest, and Mozilla. Its admin panel alone saves weeks of work. Python's dominance in data science and AI makes Django a natural choice for web apps that connect to ML models.

**Jump to it when:** You want a mature, secure, full-stack Python framework with a huge ecosystem.

---

## Flask (Python) 🟢 Mainstream

Flask is Python's Express equivalent — minimal, unopinionated, and gives you total control. You add what you need via extensions.

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/api/hello')
def hello():
    return jsonify(message='Hello World')
```

**Why it's still relevant:** Flask is the go-to for serving machine learning models as APIs. If you've trained a model in Python, Flask is the quickest path to making it accessible over HTTP.

**Jump to it when:** You want a lightweight Python server, especially for ML/AI model serving or small APIs.

---

## FastAPI (Python) 🟢 Mainstream

FastAPI is the modern Python API framework. It uses type hints to automatically validate incoming data, generate API documentation, and produce clean, safe code. It's one of the fastest Python frameworks available.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):
    name: str
    age: int

@app.post('/users')
def create_user(user: User):
    return {'created': user.name}
# FastAPI automatically validates the request body,
# returns a 422 error if types don't match,
# and generates Swagger docs at /docs
```

**Why it's still relevant:** It's become the standard for AI/ML APIs. OpenAI, Hugging Face, and most modern Python AI services use FastAPI or something built on it.

**Jump to it when:** You're building a Python API that needs to be fast, well-documented, and safe — especially for AI/ML.

---

## Laravel (PHP) 🟢 Mainstream

Laravel is the dominant PHP framework. PHP still powers over 75% of the web (WordPress, Shopify themes, etc.) and Laravel is its modern, elegant face. It's a batteries-included framework with an ORM (Eloquent), authentication, queues, mail, and a powerful CLI called Artisan.

```php
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware('auth');
```

**Why it's still relevant:** Most of the web still runs on PHP. Laravel is used by small agencies and large enterprises alike. It's one of the best developer experiences in any language.

**Jump to it when:** You're working in a PHP environment, building a traditional web application, or joining a team already on Laravel.

---

## Ruby on Rails (Ruby) 🟢 Mainstream

Rails is the original "convention over configuration" framework and invented many patterns that every other framework later copied. It has a CLI that generates models, controllers, views, migrations, and tests in seconds. GitHub, Shopify, and Airbnb were all built on Rails.

```ruby
# rails generate scaffold Post title:string body:text
# Generates the model, controller, views, routes,
# migration, and tests automatically.
```

**Why it's still relevant:** Rails remains unmatched for developer speed. Shopify alone runs millions of stores on it. The job market for Rails is healthy, especially in startups and e-commerce.

**Jump to it when:** You want to ship a product fast, especially an MVP or marketplace-type application.

---

## Spring Boot (Java) 🟢 Mainstream

Spring Boot is the standard for enterprise Java applications. Banking systems, insurance platforms, and large-scale SaaS products — if it runs Java on the backend, it's almost certainly Spring Boot.

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }
}
```

Annotating a class with `@RestController` is all it takes. Spring Boot handles the server setup, port, serialisation, and routing automatically.

**Why it's still relevant:** Java is the language of enterprise software. Spring Boot is how enterprise teams build APIs, microservices, and backend systems at scale. High-paying, stable jobs.

**Jump to it when:** You're entering enterprise software, fintech, banking, or any Java-heavy organisation.

---

## ASP.NET Core (C#) 🟢 Mainstream

Microsoft's answer to Spring Boot. ASP.NET Core is cross-platform, extremely fast, and deeply integrated with Azure. It's used in enterprise, government, and financial systems worldwide.

```csharp
app.MapGet("/hello", () => "Hello World");
```

**Why it's still relevant:** Anywhere Microsoft technology is used, ASP.NET Core is there. It consistently benchmarks among the fastest web frameworks in any language.

**Jump to it when:** You're in a Microsoft/Azure ecosystem or building enterprise-grade applications in C#.

---

## Gin (Go) 🟡 Worth Knowing

Gin is a minimalist web framework for Go. Go is a compiled language designed for high concurrency — it can handle thousands of simultaneous connections efficiently. Gin gives Go a clean HTTP framework on top of that foundation.

```go
r := gin.Default()

r.GET("/hello", func(c *gin.Context) {
    c.JSON(200, gin.H{"message": "Hello World"})
})

r.Run(":3000")
```

**Why it's still relevant:** When raw performance is the goal, Go is hard to beat. Docker, Kubernetes, and Cloudflare's infrastructure tools are written in Go. Gin is the entry point for web APIs in that world.

**Jump to it when:** You're building services that must handle extreme concurrency with minimal resource usage.

---

## Phoenix (Elixir) 🟡 Worth Knowing

Phoenix is built on Elixir, which runs on the Erlang VM — a platform famous for building systems with 99.9999% uptime. Phoenix brings those reliability and concurrency guarantees to web development.

```elixir
defmodule MyAppWeb.PageController do
  use MyAppWeb, :controller

  def index(conn, _params) do
    render(conn, :index)
  end
end
```

**Why it's still relevant:** Phoenix handles millions of concurrent WebSocket connections efficiently — something that's architecturally difficult in most other frameworks. It's the choice for real-time systems that must never go down.

**Jump to it when:** You're building real-time, fault-tolerant systems and reliability is paramount.

---

## Summary — What Should You Actually Learn?

| Priority | Framework | Why |
|---|---|---|
| **Learn first** | Express.js | Foundation of Node.js backend. Everything else builds on this understanding. |
| **Learn early** | Next.js | The React meta-framework. Most React jobs require it. |
| **Learn when ready** | NestJS | When your Express apps start getting messy and you need structure. |
| **Learn for Python** | FastAPI or Flask | FastAPI for modern APIs, Flask for ML model serving. |
| **Learn for enterprise** | Spring Boot or ASP.NET Core | Java or C# environments. High-value skills. |
| **Learn for speed** | Django or Rails | Ship full web apps fast. Proven at scale. |
| **Explore later** | Hono, Fastify, Gin | When you have a reason to care about performance at that level. |
| **Situational** | Koa, Hapi, Feathers, Encore | Solid tools, but you'll reach for them when a specific problem calls for them. |
| **Experimental** | ElysiaJS, Waku | Interesting to watch, not production-ready for most teams. |

> The honest answer: **Express.js, Next.js, and one non-JS framework** will cover 90% of what you'll ever need to build. Everything else is a tool you reach for when you've outgrown those — or when a specific problem demands it.