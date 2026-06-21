# Backend Responsibilities Over the Years
## How Backend Engineering Changed, What It Still Owns, and Where It's Going

---

A backend in 2005 meant writing everything yourself — auth flows, file handlers, email servers, payment integrations. A backend today means almost none of that. Managed services have absorbed one concern after another, and the list of things you implement from scratch keeps shrinking.
The common misread is that this makes the backend less important. What actually happened is more precise: the commodity parts got absorbed, and what remains is the part that was always irreducible — the business logic, authorization rules, and orchestration that are specific to your product and cannot be purchased anywhere.

---

## Part I — How We Got Here

### Era 1: The Monolith (2000s)

In the early 2000s, a backend was a single application that owned everything. One PHP, Rails, or Java process was responsible for authentication, file storage, HTML rendering, email delivery, payments, rate limiting, logging, and validation — all as custom code.

This was not a design philosophy. It was the only available option. There were no managed services. If you needed auth, you wrote auth.

The consequence was that backend engineering was 60% infrastructure plumbing and 40% business logic. The plumbing was repetitive, error-prone, and expensive to maintain — but unavoidable.

### Era 2: Services Split Out (2010s)

The cloud era changed the economics. Managed databases, CDNs, message queues, and compute became cheap and reliable. The backend didn't disappear — but it began outsourcing its concerns:

| Concern | Before | After |
|---|---|---|
| File storage | Server disk / custom object storage code | Managed object storage via SDK |
| Database | Self-hosted Postgres/MySQL | Managed relational databases |
| Email | SMTP, self-hosted mail servers | Transactional email APIs |
| Payments | Custom bank integration | Payment processor APIs |
| HTML rendering | Server-side templates | Client-side frameworks (SPAs) |

You still wrote a backend. You still ran an application server. But increasingly, that server's job was to orchestrate third-party services rather than implement functionality from scratch.

The mental model shifted from *"write everything"* to *"wire everything together."*

### Era 3: Backend as a Service (2020s)

The wiring itself began to be abstracted:

- **Authentication** became a hosted service — managed auth providers now handle login UI, OAuth flows, session management, and multi-factor auth. You no longer write auth implementation.
- **Databases** became API-accessible — managed cloud databases added connection pooling, auto-scaling, and in some cases full HTTP API layers with built-in permission enforcement.
- **File handling** became a processing layer — not just storage, but compute on storage. Services now handle resizing, transcoding, and format conversion on ingest. Your server never touches the file.
- **Rate limiting** became middleware — a single function call to a managed cache layer, no infrastructure required.

The pattern in each case is the same: a concern that previously required your server to implement logic, manage infrastructure, and handle failure modes was absorbed into a service you configure rather than build.

### Era 4: Intelligence as Infrastructure (Now)

The current frontier follows the same pattern at a higher level. AI capabilities — once requiring deep ML expertise to deploy — are now available as API calls. Semantic search, document analysis, content generation, and classification are becoming infrastructure concerns in the same way auth and storage did before them.

Just as "write your own auth" is no longer standard practice, "build your own ML pipeline" is rapidly becoming an unusual choice for most product use cases. A new layer of commodity capability is being abstracted into services — and as before, what remains irreducibly yours is the logic of how you use it.

---

## Part II — The Mental Model

Stop thinking about the backend as a single server. Think about it as a **layer** that enforces rules, manages state, and coordinates systems between your users and your data.

```
Client (browser, mobile app, etc.)
         ↓
Your backend layer — business logic, auth enforcement, validation, orchestration
         ↓
Infrastructure services — managed auth, database, storage, payments, email, AI
         ↓
Databases and raw data
```

When you adopt managed services, you are not removing the backend layer. You are offloading the *implementation* of certain concerns within it to specialised providers. The layer still exists. The work simply moved.

Two clarifications that follow from this:

**A managed database is still a backend.** It enforces permissions, stores state, and exposes an API. Calling it "frontend-friendly" or giving it a client SDK does not make it frontend. The backend work is still happening — you just didn't write it.

**Managed auth tells you who the user is. It does not decide what they can do.** Authentication vs. authorization is one of the most important distinctions in backend engineering. Auth services handle the first. The second is always yours.

---

## Part III — What Has Been Delegated

These concerns have been fully absorbed by managed service categories. Building them yourself is no longer standard practice — in most cases, it is a liability.

| Concern | What changed | What you retain |
|---|---|---|
| **Authentication** | Hosted auth services manage login UI, OAuth flows, sessions, tokens, and MFA. | JWT/session verification in your middleware; tying identity to authorization |
| **Database hosting** | Managed cloud databases handle connection pooling, backups, failover, and scaling. | Schema design, query logic, migrations, access rules |
| **File storage and processing** | Storage services accept uploads and process on ingest — resizing, transcoding, format conversion — without your server touching the file. | Upload endpoint, metadata persistence, references in your data model |
| **Email delivery** | Transactional email APIs replace SMTP configuration. Deliverability, bounce handling, and tracking are managed externally. | Template content, send triggers, recipient logic |
| **Payments** | Payment processors handle card data, banking integrations, and PCI compliance. | Webhook handling, business rules around who can buy what and when |
| **Rate limiting** | Managed cache layers provide rate limiting as a single middleware call. | Rate limit thresholds and rules |
| **AI/ML capabilities** | Language models, embedding services, and vector search are available as APIs. | Prompting strategy, output validation, how results integrate with your product |

---

### When Not to Delegate

Managed services are the right default, but not always. Three reasons to keep something in-house:

- **Compliance** — a healthcare app storing patient data may not be able to route it through a third-party service. A bank may be required to keep auth infrastructure within a specific region. The law decides, not you.
- **Vendor risk** — if a service you depend on for payments, auth, or storage goes down, raises prices, or shuts down, what happens to your product? For anything on the critical path, that question needs an answer before you commit.
- **It's your core product** — a recommendation engine is commodity infrastructure for most apps, but if you're building a personalisation product, that engine *is* your product. Outsourcing it means outsourcing your competitive advantage.

When in doubt: delegate infrastructure, own logic.

---

## Part IV — What the Backend Still Owns

No service replaces your business logic. This is the irreducible backend — the part that has always been the most valuable, because it is specific to your product.

**Authorization** — not "are you logged in" but "can this specific user perform this specific action on this specific resource right now." Auth services establish identity. Authorization rules are yours to define and enforce.

**Data validation** — managed services trust the data you send them. Ensuring that input is structurally valid, semantically consistent, within business rules, and safe to persist is your responsibility. Nothing upstream catches this for you.

**Business rules** — pricing logic, eligibility checks, trial period transitions, feature gating, promotional rules, approval thresholds. These are the rules that define how your product works. They cannot be purchased because they don't exist anywhere else.

**State machines** — order lifecycles, subscription transitions, approval chains, onboarding flows. Any process that moves through defined states with specific transition rules lives in your backend.

**Orchestration** — when a single user action triggers multiple downstream effects (charge payment, update inventory, send confirmation, notify a warehouse, trigger fulfillment), something has to coordinate that sequence, handle partial failures, and maintain consistency. That something is your backend.

**Domain-specific abuse prevention** — fraud patterns, manipulation of game mechanics, exploitation of pricing rules, platform-specific gaming. General-purpose rate limiting and auth don't know enough about your domain to catch these. You do.

---

### What Breaks Without This Layer

If you remove backend logic entirely and rely only on managed services:

- **Malformed input goes unchecked.** Nothing between your client and your database catches bad data.
- **Authorization collapses.** An auth service confirms user A is logged in. Nothing prevents user A from modifying user B's data unless your backend enforces that rule.
- **Business rules disappear.** A payment processor can charge a card. It cannot decide whether this user is currently eligible to purchase this product.
- **Orchestration fails.** No external service knows that placing an order should also update inventory, trigger fulfillment, and send a confirmation. That coordination is yours.

The responsibility doesn't disappear when you remove the server. It becomes a gap — one that will be exploited.

---

## Conclusion

Every era of backend development has followed the same pattern: concerns that were once implemented from scratch became services that were configured instead. Custom SMTP servers became email APIs. Self-hosted databases became managed cloud databases. Hand-written auth became hosted auth providers. Now, ML infrastructure is becoming an API call.

The pattern will continue. What is commodity today was bespoke in 2005.

What has never moved is the business logic that defines your product — authorization rules, data validation, pricing logic, state machines, orchestration, domain-specific constraints. These are not infrastructure. They are product. No service can absorb them because they don't exist anywhere except in your understanding of what you are building.

The backend you write today is narrower than it was in 2005. But what remains is the part that was always the hardest and most valuable.

---

*The dividing line isn't "frontend vs backend." It's "commodity vs product." Delegate the first. Own the second. Know the difference.*