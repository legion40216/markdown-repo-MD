# 🌐 Web Servers

> **Core idea:** A web server is software (or hardware running that software) that listens for incoming network requests and sends back responses — HTML pages, JSON data, files, etc.

---

## "Web Server" Means Two Different Things

The term gets used for both the machine and the software running on it.

The **machine** (physical or virtual) is just a computer in a data center. It has a CPU, RAM, a disk, and a network card. By itself it knows nothing about HTTP.

The **software** — Nginx, Apache, Caddy — is the program running on that machine that actually speaks HTTP. It's what listens on port 80, parses incoming requests, and sends responses back.

| Term | Meaning |
|---|---|
| **Web server software / HTTP server** | The program — Nginx, Apache, Caddy |
| **Web server hardware** | The physical or virtual machine hosting it |

> When people say *"deploy to a server"* they mean the machine. When they say *"configure your web server"* they mean the software. These are different things.

---

## The Two Layers In The Web Server Software

The infrastructure layer (Nginx, Apache) handles raw HTTP connections, SSL termination, and static file serving. It deliberately knows nothing about your application logic.

The application layer runs your actual code — routing requests, executing middleware, managing authentication, and talking to databases.

> Now days these have merged with runtime solution like nodejs.

---

## What a Web Server Actually Does

At its core, every web server follows this pipeline:

```
1. Opens a socket and listens on a port
2. Accepts an incoming TCP connection
3. Reads raw bytes off the wire
4. Parses those bytes into an HTTP request
         ↓ server hands off to your code
5. Figures out what to do with it
6. Writes an HTTP response back
         ↑ server takes back over
7. Closes (or keeps alive) the connection
```

### What the Web Server Does Before Your Code Runs

**Sockets**

The very foundation. A socket is an API the OS provides to send and receive bytes over a network. Every web server starts up the same way:

```
socket() → bind() → listen() → accept()
```

Your application never touches this. The connection is already established before you see anything.

**HTTP Parsing**

The server calls `recv()` on the socket to pull bytes into a buffer, then hands them to a parser. Raw bytes arrive looking like:

```
GET /home HTTP/1.1
Host: example.com
Accept: text/html
```

The server parses this into structured data — method, path, headers, body — before handing it to your code. By the time your handler runs, you're working with `req.method` and `req.path`, never raw bytes.

**Static File Serving**

For static assets — images, CSS, JS files — the web server handles the response entirely. Your application is never involved. Nginx uses OS-level tricks to do this fast:

- `sendfile()` syscall — the kernel copies the file directly to the socket, bypassing your application entirely
- `mmap()` — maps a file into memory so repeated requests skip the disk

This is why Nginx serves static files far faster than reading them manually in application code.

**Connection Management**

HTTP/1.1 keep-alive reuses the same TCP connection for multiple requests, avoiding the cost of reopening a socket on every round trip. HTTP/2 goes further — it multiplexes several requests over a *single* connection simultaneously, so a page's CSS, JS, and images all arrive in parallel without head-of-line blocking. Both are handled transparently at the web server layer. Your application code never thinks about them.

### What Your Code Handles

By this point the request is a clean, structured object. Now it's yours.

**Middleware Pipeline**

The request flows through a chain of functions you define:

```
Request → Auth check → Rate limiter → Logger → Router → Handler → Response
```

In Hono this looks like:

```ts
app.use(logger())
app.use(authMiddleware())
app.get('/rooms', getRooms)
```

Each function either passes the request forward or short-circuits with a response — a failed auth check never reaches the router.

**Response Serialization**

Your handler returns a structured object — a status code, headers, a body. The server turns that back into raw bytes and pushes them down the socket. You never construct `HTTP/1.1 200 OK\r\n` yourself — the framework handles that translation, the same way it translated incoming bytes into `req.method`.

---

## Popular Web Server Software

| Software | Known For |
|---|---|
| **Nginx** | Very fast, event-driven, widely used as a reverse proxy |
| **Apache** | One of the oldest; highly configurable |
| **Caddy** | Modern, automatic HTTPS, simple config |
| **IIS** | Microsoft's server, built into Windows Server |

---

## The Evolution of Web servers

```
1990s   Apache       → serve static files over HTTP
2000s   Nginx        → handle massive concurrent traffic, reverse proxy
2010s   Node.js      → app server in JS using the same event model
2020s   Edge/Bun/Deno → app + server merged, runs at the CDN edge
```
> Each generation solved the previous generation's bottleneck.

**CGI (late 90s):** Apache spawned a new process per request for dynamic content. Simple but very slow.

**The C10K Problem (1999):** A webpage by engineer Dan Kegel posed the question: *"Why can't a server handle 10,000 concurrent connections?"* Apache's one-thread-per-request model collapsed under load. This led to the event-driven architecture that Nginx (2004) was built on.

**Node.js (2009):** Brought the event loop to the application layer, collapsing web server and app server into one runtime.

**Edge servers (2020s):** Cloudflare Workers, Vercel Edge — your code runs *inside* the CDN, physically close to the user. Web server, app server, and CDN merge into one.

---

## Web Server Frameworks

A Web server **framework** is structured tooling built on top of whatever HTTP solution the runtime provides, and many languages have been built on top of web server solutions to provide their own strengths to differnt problems.

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

---

## The Concurrency Model

Every runtime makes a choice about how it handles many simultaneous requests. That choice ripples into every line of code you write.

| Model | How it works | Used by |
|---|---|---|
| Multi-process | Run many copies of the app | PHP, Ruby, Python |
| Multi-threaded | Assign a thread per request | Java, .NET |
| Event loop | One thread; parks requests during I/O | Nginx, Node.js |
| Green threads | OS-scheduled lightweight threads | Go (goroutines) |

You don't configure the concurrency model — your runtime picks it. But you feel it everywhere:

- A **Node.js** app must write `async`/`await` code because the event loop demands it — a blocking call stalls the entire server
- A **Python or Ruby** app can write blocking code freely because the worker pool absorbs it
- A **Go** app can write synchronous-looking code because goroutines are cheap enough that blocking one barely matters

The model is invisible in config, but visible in every line you write.

---

### The Direction of Travel

The industry has consistently moved toward fewer layers:

```
PHP era:        Language embedded in web server
Ruby/Python:    Language runs separately; web server proxies to it
Node/Go:        Language IS the web server
Edge era:       App + server + CDN merged into one
```

Each step eliminated a layer that existed to work around a previous limitation. CGI was slow, so PHP embedded into Apache. Embedding was too coupled, so Ruby and Python standardized a clean handoff. Blocking I/O was expensive, so the event loop removed it. Separate servers were complex, so the edge merged them.