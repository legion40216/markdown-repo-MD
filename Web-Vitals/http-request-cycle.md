# The HTTP Request Cycle: What Really Happens When You Visit a Website

Every time you type a URL and press Enter, your browser silently executes a sophisticated multi-phase operation — spanning DNS lookups, cryptographic handshakes, packet routing across continents, and server-side logic — all in under a second. Most developers interact with only the topmost slice of this process. This article walks through the entire lifecycle, layer by layer, so you understand not just *what* happens but *why* each piece exists.

---

## The Big Picture

Before diving into specifics, it helps to understand the shape of the journey. A single HTTP request goes through eight distinct phases:

1. **URL parsing & cache check** — breaking down the URL and checking if the response is already stored locally
2. **DNS resolution** — translating a domain name into an IP address
3. **TCP handshake** — opening a reliable connection to the server
4. **TLS handshake** — negotiating encryption (HTTPS only)
5. **HTTP request** — sending your actual data down through the OSI stack
6. **Server processing** — unwrapping layers, running app logic, querying databases
7. **HTTP response** — the server's reply traveling back the same path in reverse
8. **Browser rendering** — parsing and painting the response (if HTML)

None of these phases is optional or redundant. Each solves a specific problem that the others don't touch.

---

## Phase 1 — URL Parsing & Cache Check

Before anything hits the network, the browser does two things: parse the URL and check whether it already has a valid answer stored locally.

### Parsing the URL

The browser splits the URL into its components: protocol (`https`), hostname (`example.com`), path (`/page`), and query string. This tells it which server to contact, over which protocol, and what resource to request.

### Checking the caches

The browser then checks several layers of cache, in order:

- **Browser memory cache** — a short-lived in-memory store for resources fetched during the current session
- **Service Worker cache** — if a Service Worker is registered for this origin, it can intercept the request entirely and respond from its own cache, without any network activity
- **Disk cache** — persisted across browser sessions; checked against `Cache-Control` and `Expires` headers from the original response

If a valid cached response exists and hasn't expired, **the entire lifecycle ends here**. No DNS lookup, no TCP connection, no server involvement. This is why caching is the single highest-leverage performance optimization available.

If the cache is stale but has an `ETag` or `Last-Modified` header, the browser may issue a **conditional request** — asking the server "has this changed?" If not, the server returns `304 Not Modified` with no body, and the browser reuses its cached copy.

---

## Phase 2 — DNS Resolution

The internet routes traffic using IP addresses — numeric identifiers like `93.184.216.34`. But humans use domain names like `example.com`. DNS (Domain Name System) is the translation layer between the two.

When you visit a site, your browser doesn't know the IP address. It must ask. The lookup cascades through a hierarchy of resolvers:

1. **Browser cache** — the browser remembers recent lookups. Sub-millisecond if it hits.
2. **OS resolver** — your operating system maintains its own DNS cache.
3. **ISP resolver** — your internet provider's DNS server, shared across thousands of users.
4. **Authoritative nameserver** — the definitive source of truth for that domain. Maintained by whoever owns the domain.

Each layer caches the result for a period defined by the DNS record's **TTL** (Time To Live). A short TTL means frequent lookups and faster propagation of changes. A long TTL means faster resolution for frequent visitors but slower updates when DNS changes.

An important detail: DNS resolution happens *before* any TCP connection is made. At this stage, no data has been exchanged with the web server at all.

---

## Phase 3 — TCP Handshake

With an IP address in hand, the browser can now open a connection. The internet's core transport protocol — TCP (Transmission Control Protocol) — guarantees reliable, ordered delivery of data. But before data can flow, both sides must agree to communicate. This agreement is the famous **three-way handshake**:

- **SYN** — the client sends a synchronize packet: "I want to connect. Here's my starting sequence number."
- **SYN-ACK** — the server acknowledges and responds with its own sequence number.
- **ACK** — the client acknowledges the server's response. Connection is now open.

These sequence numbers are how TCP keeps track of all the individual packets being exchanged. If a packet is dropped or arrives out of order, TCP detects it and requests a retransmission. This reliability guarantee costs roughly one round-trip of latency — a trade-off that protocols like QUIC (used in HTTP/3) are designed to reduce.

Crucially, the TCP handshake is a **network-level operation**. No HTTP, no cookies, no application data of any kind is exchanged here. It's purely infrastructure.

---

## Phase 4 — The TLS Handshake

If you're using HTTPS (which is now the default for virtually all web traffic), a second handshake takes place immediately after TCP. This is the TLS (Transport Layer Security) handshake, and it has one job: establish an encrypted channel before any application data is sent.

### How the handshake works

**Step 1: Client Hello.** The browser sends a message listing the TLS versions and cipher suites it supports — essentially its encryption capabilities.

**Step 2: Server certificate.** The server responds with its SSL/TLS certificate, issued by a Certificate Authority (CA) like Let's Encrypt or DigiCert. This certificate contains the server's public key and is signed by the CA to prove authenticity.

**Step 3: Certificate verification.** The browser checks three things: Is this certificate signed by a CA it trusts? Does the domain name in the certificate match the one you're visiting? Has the certificate expired? If any check fails, you see the red padlock warning.

**Step 4: Key exchange.** Here's where cryptographic magic happens. Both sides run a mathematical process — typically Diffie-Hellman key exchange — that allows them to independently derive the same shared session key *without ever transmitting it*. An eavesdropper who intercepts every packet of the handshake still cannot compute the session key. This property is called **forward secrecy**.

After the handshake, every byte of the HTTP conversation is encrypted with that session key. Routers in between see nothing but ciphertext.

### Who does what

Understanding TLS responsibility is practically useful for developers:

| Party | Responsibility |
|---|---|
| Browser | Initiates TLS, verifies the certificate, encrypts outgoing requests |
| OS networking stack | Manages the underlying TCP socket |
| Server (nginx, Apache) | Holds the private key, performs TLS termination, decrypts incoming data |
| Your app code | Never touches encryption — receives a fully decrypted HTTP request |

This last point matters: if you're writing Express, Django, Rails, or any web framework, TLS is completely invisible. By the time a request reaches your route handler, nginx or the runtime has already decrypted it. You receive a plain `req` object with headers, body, and cookies. The abstraction is total.

---

## Phase 5 — The HTTP Request and the OSI Model

Now the browser can finally send its HTTP request. A typical request looks like:

```
GET /api/users/42 HTTP/1.1
Host: example.com
Accept: application/json
Accept-Encoding: gzip, br
Authorization: Bearer <token>
Connection: keep-alive
User-Agent: Mozilla/5.0 ...
```

The key parts of the message are:

- **Request line** — the method (`GET`, `POST`, etc.), the path, and the protocol version
- **Headers** — metadata about the client, accepted formats, auth tokens, cookies
- **Body** — present for `POST`/`PUT`/`PATCH`; contains JSON, form data, file uploads, etc.

But the request doesn't travel as a single message. It gets wrapped in successive layers of protocol headers — like envelopes inside envelopes — as it descends through the networking stack. This layered model is formalized as the **OSI Model**, which defines seven layers of abstraction.

### The seven layers

**Layer 7 — Application.** This is your actual HTTP message: the method, the path, headers (including `Cookie: session_id=abc123`), and the request body. Cookies live here. They are simply a string in the `Cookie` header — nothing more architecturally complex than that.

**Layer 6 — Presentation.** TLS encryption and decryption happen at this layer. The plaintext HTTP from Layer 7 becomes opaque ciphertext before it descends further.

**Layer 5 — Session.** Manages the open connection and coordinates dialogue between client and server.

**Layer 4 — Transport.** TCP wraps the data in a segment that includes port numbers (e.g., 443 for HTTPS) and sequence numbers for reliable ordering. This is the layer where your message becomes a series of packets tracked across the network.

**Layer 3 — Network.** IP wraps the TCP segment in a packet containing source and destination IP addresses. Routers operate at this layer. They strip and re-wrap L1–L2 headers at each hop, making forwarding decisions based solely on the IP address. Your request may traverse 10–20 routers.

**Layer 2 — Data Link.** Each local network hop wraps the IP packet in an Ethernet frame with MAC (hardware) addresses. MAC addresses only apply to the current segment — they change at every router hop, unlike IP addresses which remain constant end-to-end.

**Layer 1 — Physical.** The actual transmission of bits as electrical signals, pulses of light through fiber, or radio waves in the case of Wi-Fi.

### The key insight about the OSI model

Each layer knows nothing about the layers above or below it in terms of content. A router at Layer 3 doesn't know or care that the IP packet it's forwarding contains HTTP — it just routes based on the destination IP. TCP at Layer 4 doesn't know the payload is a cookie string — it just ensures reliable delivery. This strict separation of concerns is what makes the internet composable and extensible.

---

## Phase 6 — Server-Side Processing

When the request arrives at the server, the process runs in reverse. The server unwraps each layer, verifies checksums, reassembles TCP segments, decrypts TLS, and delivers the HTTP message upward to the application stack.

The server-side stack typically consists of several distinct components, each handling a specific concern:

### Load balancer

In production environments, the first thing a request hits is usually a load balancer. It distributes traffic across multiple server instances and often handles TLS termination so that backend servers receive plain HTTP internally. This centralizes certificate management and reduces CPU overhead on app servers.

### Web server — nginx or Apache

The web server handles static assets (images, stylesheets, JavaScript files) directly without touching the application. For dynamic requests, it proxies the traffic to the app. Nginx is also where TLS termination often occurs if there's no separate load balancer.

### Middleware and authentication

This is where the `Cookie` header becomes meaningful. Middleware reads the session ID from the cookie, looks it up in a cache (Redis is common) or database, and either attaches user context to the request or returns a 401 Unauthorized response. Authentication logic lives here, not inside the framework — which is why it applies across routes automatically.

### Application handler

Your actual business logic. In Express, Django, Rails, FastAPI, or whatever framework you use, this is the route handler that receives the request. By this point, everything is fully decoded: cookies are a parsed object, headers are a dictionary, the body is deserialized JSON or form data. The handler queries a database, calls third-party APIs, builds a response.

### Database

Persistent data lives here. Notably, this is where passwords are stored — but not in plaintext, and not protected by TLS. TLS protected the password *on the wire* while the user was logging in. Once it's in the database, a different threat applies: a database breach. This is why passwords are hashed with bcrypt or Argon2. These algorithms are designed to be computationally expensive to reverse, so that stolen hashes are not useful without extreme brute-force effort.

TLS and bcrypt are not redundant — they defend against entirely different attacks. TLS defends against interception on the network. bcrypt defends against database exfiltration.

---

## Phase 7 — The HTTP Response

The response travels the same path in reverse. The server's app code produces an HTTP response — status code, headers, body — which flows back through the server stack, gets encrypted by TLS, descends the OSI layers, crosses the network, and arrives at the browser.

A typical response looks like:

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Encoding: gzip
Cache-Control: max-age=3600
ETag: "abc123"
Set-Cookie: session=xyz; HttpOnly; Secure

{ "id": 42, "name": "Alice" }
```

The key parts are:

- **Status line** — the protocol version and status code (`200 OK`, `404 Not Found`, `301 Moved Permanently`, etc.)
- **Response headers** — caching directives, content type, CORS headers, and any cookies to set
- **Body** — HTML, JSON, binary file data, etc.

If the server wants to create or update a cookie — for example, after a successful login — it includes a `Set-Cookie` header in the response:

```
Set-Cookie: session_id=abc123xyz; HttpOnly; Secure; SameSite=Strict; Max-Age=86400
```

From that point forward, the browser automatically attaches that cookie to every matching request. No JavaScript required. The browser's networking stack handles it transparently.

---

## Phase 8 — Connection Handling

Once the response is received, the browser decides what to do with the TCP connection.

With `Connection: keep-alive` (the default in HTTP/1.1), the connection is kept open and reused for subsequent requests to the same server — avoiding the cost of a new TCP and TLS handshake each time.

**HTTP/2** goes further: it multiplexes multiple requests over a single TCP connection simultaneously, eliminating head-of-line blocking at the HTTP layer. Several assets can be in-flight at once without waiting for each other.

**HTTP/3** replaces TCP entirely with **QUIC** — a UDP-based transport protocol developed by Google. QUIC eliminates TCP's head-of-line blocking at the transport layer as well, and makes connection setup faster on unreliable networks by combining the transport and TLS handshakes into a single round trip.

---

## Phase 9 — Browser Rendering (if HTML)

If the response body is an HTML document, the browser's work is only beginning. Rendering involves several sequential sub-phases:

1. **Parse HTML → build the DOM** — the browser constructs a Document Object Model tree representing the page structure
2. **Fetch linked resources** — `<link>`, `<script>`, and `<img>` tags trigger new HTTP cycles, each going through the full pipeline above
3. **Parse CSS → build the CSSOM** — stylesheets are parsed into a separate CSS Object Model tree
4. **Combine DOM + CSSOM → Render Tree** — only visible elements and their computed styles are included
5. **Layout** — the browser calculates the exact position and dimensions of every element
6. **Paint → Composite → pixels** — elements are drawn to layers, composited, and finally appear on screen

This is why optimizing page load isn't just about the first request — it's about the cascade of subsequent requests for CSS, JavaScript, fonts, and images that HTML triggers.

---

## Deep Dive: Cookies

Cookies are frequently misunderstood because they sound like a browser-specific mechanism. In reality, they are purely an application-layer (L7) construct. A cookie is nothing more than a key-value string that:

1. The server sends via `Set-Cookie` header in a response
2. The browser stores locally
3. The browser attaches as a `Cookie` header on every subsequent matching request

The security properties of a cookie depend entirely on the attributes set when it's created:

- **HttpOnly** — JavaScript cannot access this cookie. Prevents XSS attacks from stealing session tokens via `document.cookie`.
- **Secure** — the browser only sends this cookie over HTTPS connections, never HTTP.
- **SameSite=Strict** — the browser never sends this cookie in cross-site requests. Prevents CSRF attacks where a malicious site tricks your browser into making authenticated requests.
- **Expires / Max-Age** — when the cookie should be deleted. Without these, it's a session cookie and disappears when the browser closes.
- **Domain / Path** — restricts which URLs the cookie is sent to.

Cookies ride passively through every layer below L7. Not a single router, switch, or TCP implementation reads, modifies, or even acknowledges their existence. They are just bytes in a payload being transported from point A to point B.

---

## Summary Timeline

```
URL parse + cache check  ~0ms        (cache hit: cycle ends here)
DNS lookup               ~20–120ms   (cached: ~0ms)
TCP handshake            ~1 RTT      (~10–100ms)
TLS handshake            ~1 RTT      (TLS 1.3)
Request transit          ~RTT/2
Server processing        ~1–500ms+
Response transit         ~RTT/2
Browser rendering        ~10–200ms+
─────────────────────────────────────
Total                    ~50ms–1s+ for a typical page load
```

The biggest levers for performance are: cache utilization, DNS TTL tuning, connection reuse and HTTP/2 multiplexing, CDN proximity, server-side latency, and response size with compression.

---

## Putting It All Together

The lifecycle of a web request is best understood as a series of nested abstractions, each solving one problem:

- **URL parsing** solves the input problem — understanding what was requested before doing any work
- **Caching** solves the redundancy problem — avoiding network trips for responses that haven't changed
- **DNS** solves the human-readability problem — names instead of numbers
- **TCP** solves the reliability problem — guaranteed delivery across an unreliable network
- **TLS** solves the privacy and authenticity problem — encrypted, verified communication
- **OSI layers** solve the modularity problem — each layer can evolve independently
- **HTTP** solves the application communication problem — a standard language for clients and servers
- **Cookies** solve the statelessness problem — HTTP is stateless by design, so cookies carry identity across requests
- **Server stack** solves the scalability and separation-of-concerns problem — each component handles what it's good at
- **Browser rendering** solves the presentation problem — turning bytes into pixels

What makes this system remarkable is how thoroughly each layer hides its complexity from the ones above it. An Express route handler knows nothing about TCP sequence numbers. A router knows nothing about HTTP. TLS is invisible to your app code. These abstractions are what allow web development to be approachable — you can build sophisticated applications without understanding every layer in depth.

But when something goes wrong — a TLS certificate expires, a cookie gets misconfigured and allows session hijacking, a DNS misconfiguration sends traffic to the wrong server — understanding the full stack is what separates a developer who can debug it from one who cannot.

---

## Summary Reference

| Component | Layer | Problem it solves |
|---|---|---|
| URL parsing | Pre-connection | Breaking down the request URL |
| Browser / disk cache | Pre-connection | Avoiding unnecessary network requests |
| Service Worker | Pre-connection | Programmable request interception and caching |
| DNS | Pre-connection | Name → IP address translation |
| TCP handshake | Network (L4) | Reliable, ordered connection |
| TLS handshake | Presentation (L6) | Encryption + server authentication |
| IP routing | Network (L3) | Hop-by-hop packet forwarding |
| HTTP | Application (L7) | Client-server communication protocol |
| Cookie header | Application (L7) | Session state across stateless requests |
| nginx / web server | Server-side | Static files, TLS termination, proxying |
| Middleware | Server-side | Auth, logging, request transformation |
| bcrypt | Database level | Protection against stored credential theft |
| HTTP/2 + HTTP/3 | Transport | Multiplexing, reduced handshake overhead |
| Browser rendering | Client-side | Parsing and painting the HTTP response |

---

*Understanding this lifecycle won't make you write better `for` loops. But it will make you a better debugger, a better security thinker, and a developer who can reason confidently about what happens between the keyboard and the response — the invisible machinery that every web application runs on.*