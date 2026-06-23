# 🌐 Web Servers

> **Core idea:** A web server is software (or hardware running that software) that listens for incoming network requests and sends back responses — HTML pages, JSON data, files, etc.

---

## "Web Server" Means Two Different Things

The term gets used for both the machine and the software running on it.

The **machine** (physical or virtual) is just a computer in a data center. It has a CPU, RAM, a disk, and a network card. By itself it knows nothing about HTTP.

The **software** — Nginx, Apache, Caddy — is the program running on that machine that actually speaks HTTP. It's what listens on port 80, parses incoming requests, and sends responses back.

> You can turn your laptop into a web server just by running web server software locally on your own machine.

| Term | Meaning |
|---|---|
| **Web server software / HTTP server** | The program — Nginx, Apache, Caddy |
| **Web server hardware** | The physical or virtual machine hosting it |

> When people say *"deploy to a server"* they mean the machine. When they say *"configure your web server"* they mean the software. These are different things.

---

## What a Web Server Actually Does

At its core, every web server follows this pipeline:

```
1. Opens a socket and listens on a port
2. Accepts an incoming TCP connection
3. Reads raw bytes off the wire
4. Parses those bytes into an HTTP request
         ↓ server hands off to your application
5. Figures out what to do with it
6. Writes an HTTP response back
         ↑ server takes back over
7. Closes (or keeps alive) the connection
```

### Sockets

The very foundation. A socket is an API the OS provides to send and receive bytes over a network. Every web server starts up the same way:

```
socket() → bind() → listen() → accept()
```

Your application never touches this. The connection is already established before you see anything.

### HTTP Parsing

The server calls `recv()` on the socket to pull bytes into a buffer, then hands them to a parser. Raw bytes arrive looking like:

```
GET /home HTTP/1.1
Host: example.com
Accept: text/html
```

The server parses this into structured data — method, path, headers, body — before handing it to your application. By the time your handler runs, you're working with `req.method` and `req.path`, never raw bytes.

### Static File Serving

For static assets — images, CSS, JS files — the web server handles the response entirely. Your application is never involved. Nginx uses OS-level tricks to do this fast:

- `sendfile()` syscall — the kernel copies the file directly to the socket, bypassing your application entirely
- `mmap()` — maps a file into memory so repeated requests skip the disk

This is why Nginx serves static files far faster than reading them manually in application code.

### Connection Management

HTTP/1.1 keep-alive reuses the same TCP connection for multiple requests, avoiding the cost of reopening a socket on every round trip. HTTP/2 goes further — it multiplexes several requests over a *single* connection simultaneously, so a page's CSS, JS, and images all arrive in parallel without head-of-line blocking. Both are handled transparently at the web server layer. Your application code never thinks about them.

---

## Popular Web Server Software

| Software | Known For |
|---|---|
| **Nginx** | Very fast, event-driven, widely used as a reverse proxy |
| **Apache** | One of the oldest; highly configurable |
| **Caddy** | Modern, automatic HTTPS, simple config |
| **IIS** | Microsoft's server, built into Windows Server |

---

## The Evolution of Web Servers

```
1990s   Apache       → serve static files over HTTP
2000s   Nginx        → handle massive concurrent traffic, reverse proxy
2010s   Node.js http → a runtime module that lets you build a server in JS
2020s   Edge/Bun/Deno → server runs at the CDN edge, close to the user
```

> Each generation solved the previous generation's bottleneck.

**CGI (late 90s):** Apache spawned a new process per request for dynamic content. Simple but very slow.

**The C10K Problem (1999):** A webpage by engineer Dan Kegel posed the question: *"Why can't a server handle 10,000 concurrent connections?"* Apache's one-thread-per-request model collapsed under load. This led to the event-driven architecture that Nginx (2004) was built on.

**Node.js (2009):** A JavaScript runtime that ships with an `http` module, giving you low-level primitives to build a server yourself. It is not a web server out of the box — it becomes one only when you write code that makes it listen for requests.

**Edge servers (2020s):** Cloudflare Workers, Vercel Edge — your code runs *inside* the CDN, physically close to the user. Web server and CDN merge into one.

---

## The Concurrency Model

Every web server makes a choice about how it handles many simultaneous requests. That choice shapes everything built on top of it.

| Model | How it works | Used by |
|---|---|---|
| Multi-process | Run many copies of the app | Apache (prefork) |
| Multi-threaded | Assign a thread per request | IIS, Apache (worker) |
| Event loop | One thread; parks requests during I/O | Nginx, Node.js http |
| Green threads | Lightweight OS-scheduled threads | Go |

The event loop model is why Nginx can handle tens of thousands of concurrent connections without spawning thousands of threads — it never blocks waiting on I/O, it just parks a request and moves on to the next one.

---

## The Direction of Travel

The industry has consistently moved toward fewer layers:

```
PHP era:     Language embedded in web server
Ruby/Python: Language runs separately; web server proxies to it
Node.js:     Runtime provides the primitives to build a server in JS
Edge era:    Server + CDN merged into one, runs close to the user
```

Each step eliminated a layer that existed to work around a previous limitation. CGI was slow, so PHP embedded into Apache. Embedding was too coupled, so Ruby and Python standardized a clean handoff. Separate servers were complex, so the edge merged them.