# The 6-Month Backend Engineer Roadmap
### From capable builder → hire-ready junior backend engineer

**Starting point (confirmed):** You're comfortable with Node.js/Express, MongoDB, JWT auth, sessions, middleware architecture, REST design, OAuth flows, Nodemailer, and error handling. You don't need beginner content. This roadmap only covers what's between you and a hiring manager's "yes."

**Target:** Employable as a backend intern / junior backend developer within 6 months.

---

## 1. How This Document Works

Three rules govern every module below, and they exist specifically so you don't end up overwhelmed:

1. **One flagship project, the whole way through.** You are not building 8 separate apps to learn 8 topics. You are building **one** production-grade backend and bolting on a new capability every module. By month 6, that one project *is* your portfolio piece — and every line on your resume maps to a real, working feature you can explain in depth.
2. **Modules are sequential, not parallel.** Don't start Module 3 while "still meaning to circle back" to Module 2. Each module has an **exit criteria** checklist — when you can tick every box, you move on. Not before, not after agonizing over more depth than a junior role needs.
3. **Depth is capped at "junior-hire-level," on purpose.** For each topic I'm telling you the depth that gets you *hired*, not the depth of a 10-year staff engineer. Going deeper than this on your own time is fine later — but chasing rabbit holes now is the single most common way self-taught developers burn 6 months and end up with nothing to show for it.

---

## 2. Time Budget Assumption

This plan assumes **~12–15 focused hours/week** (roughly 2 hours on weekdays + a longer weekend session). That's a sustainable pace for someone learning alongside other commitments, not a burnout pace.

- **More time available?** Don't skip ahead to later modules — go *deeper* on the current module's stretch goals (marked 🔹 below) instead.
- **Less time some weeks?** Let the calendar dates below slide. The module order and exit criteria matter far more than hitting exact dates.

---

## 3. Your Flagship Project: "TeamForge"

A multi-tenant **team & project operations backend** — think of it as the API layer behind a Jira/Linear/Asana-style tool. (Rename it whatever you like — the domain complexity is what matters, not the branding.)

**Why this domain specifically:** it naturally forces every skill in this roadmap to exist for a real reason, not as a contrived exercise:

| Entity | Forces you to practice |
|---|---|
| Organizations → Teams → Projects → Tasks | Relational modeling, normalization, one-to-many & many-to-many relationships |
| Users with Org/Team roles (Owner/Admin/Member/Viewer) | RBAC, authorization logic, JWT claims design |
| Comments, Attachments, Activity Log | Audit trails, pagination, filtering |
| Notifications (assignment, mentions, due-date digests) | Background jobs, queues, Nodemailer at scale |
| Dashboard stats (tasks per status, overdue count) | Caching, query optimization, aggregation |
| The whole thing, containerized and shipped | Docker, CI/CD, reverse proxies |

You'll design the schema in Module 2 and never rebuild it from scratch — only extend it.

---

## 4. Roadmap at a Glance

| # | Module | Weeks | Approx. Dates | Core Deliverable |
|---|---|---|---|---|
| 1 | TypeScript for Backend Engineers | 3 | Jul 14 – Aug 3 | TeamForge scaffolded in strict-mode TS |
| 2 | Database Engineering (SQL/Postgres + Mongo deepening) | 4 | Aug 4 – Aug 31 | Normalized Postgres schema + first transaction |
| 3 | Production-Grade API Development | 5 | Sep 1 – Oct 5 | Hardened, documented, queue-backed API |
| 4 | Testing Engineering | 3 | Oct 6 – Oct 26 | Unit + integration test suite with CI-ready coverage |
| 5 | Caching & Performance | 2 | Oct 27 – Nov 9 | Redis caching + HTTP caching on live endpoints |
| 6 | Deployment & DevOps | 4 | Nov 10 – Dec 7 | Dockerized app, CI/CD pipeline, HTTPS-served |
| 7 | System Design + AI-Assisted Engineering | 3 | Dec 8 – Dec 28 | System design doc + 3 practiced whiteboard problems |
| 8 | Capstone Polish + Interview Prep | 2 | Dec 29 – Jan 11 | Portfolio-ready repo, resume bullets, mock interviews |

Modules 7–8 land over the winter holidays on purpose — they're the lightest, most conceptual/consolidation-heavy modules, so a lower-intensity couple of weeks won't derail you.

---

## 5. Module 1 — TypeScript for Backend Engineers
**Weeks 1–3**

**Why it matters:** Most professional Node job postings now list TypeScript as required or strongly preferred. More importantly, it's a maturity signal — it tells an interviewer you think about contracts and failure modes before you ship, not after.

**Core concepts checklist**
- [ ] Interfaces vs. type aliases, literal types, enums vs. union literals
- [ ] Typing functions: parameters, return types, overloads
- [ ] Generics: generic functions, generic classes, constraints (`extends`), defaults
- [ ] Utility types: `Partial`, `Pick`, `Omit`, `Record`, `Readonly`, `ReturnType`, `Awaited`
- [ ] Discriminated unions + type guards (the pattern you'll use constantly for API responses/errors)
- [ ] Basic conditional & mapped types (enough to read library type definitions, not to author complex ones)
- [ ] Type-safe Express: typing `Request`/`Response`/`NextFunction`, extending `Request` with `req.user`, typed middleware
- [ ] Runtime validation with type inference (Zod) — this is the bridge between "TypeScript looks safe" and "TypeScript is actually safe," since TS types disappear at runtime
- [ ] Production `tsconfig.json`: `strict: true`, `noImplicitAny`, module resolution, path aliases
- [ ] Incremental JS→TS migration strategy (`allowJs`, `checkJs`, converting file-by-file without a big-bang rewrite)

**Build this:** Scaffold TeamForge as a TypeScript-first Express app in strict mode. Define your core domain types (User, Organization, Task, etc.) as TS interfaces before you have a database — this forces you to think about your data model early. Add Zod schemas + inferred types for at least 3 request bodies (e.g., create-task, create-user, update-task-status).

**Exit criteria — you're ready to move on when you can:**
- [ ] Write a generic `Repository<T>`-style helper function without looking it up
- [ ] Explain in one sentence why `any` defeats the purpose of using TypeScript at all
- [ ] Explain the difference between a type that exists only at compile time vs. one enforced at runtime (and why Zod bridges that gap)

**Reference points:** TypeScript Handbook (official docs), Zod docs.

---

## 6. Module 2 — Database Engineering
**Weeks 4–7**

**Why it matters:** SQL comes up in nearly every backend interview, in some form — a query to write, a schema to critique, or a "how would you model this" question. Schema design and indexing decisions are also how interviewers distinguish "can write CRUD" from "understands data."

### 6.1 Track A: SQL / PostgreSQL (the primary new skill)

- [ ] SQL fundamentals: `SELECT`, `JOIN` (inner/left/right), `GROUP BY`, subqueries, CTEs (`WITH`)
- [ ] Schema design & normalization (1NF → 3NF), and *when* to deliberately denormalize
- [ ] Relationships: one-to-many, many-to-many via join tables, foreign keys, `ON DELETE CASCADE`/`SET NULL`
- [ ] Indexing: B-tree basics, composite indexes, reading `EXPLAIN ANALYZE`, why indexes slow down writes
- [ ] Transactions: ACID properties, isolation levels (at least know `READ COMMITTED` vs `SERIALIZABLE` exist), deadlocks
- [ ] Query optimization: the N+1 query problem, eager vs. lazy loading, connection pooling
- [ ] Scaling fundamentals (conceptual): read replicas, vertical vs. horizontal scaling, sharding — you need to *discuss* these, not implement them at junior level
- [ ] Tooling progression: write raw SQL via `pg` first (so the fundamentals aren't hidden), *then* adopt a typed ORM/query builder (Prisma pairs naturally with Module 1's TypeScript)

### 6.2 Track B: MongoDB Deepening (reinforcing what you already know)

Since you already have working MongoDB experience, this track is short and targeted — the goal is depth, not repetition of CRUD basics:
- [ ] Aggregation pipeline mastery (`$match`, `$group`, `$lookup`, `$facet`)
- [ ] Compound and text indexes
- [ ] Schema design patterns: embedding vs. referencing, and the tradeoffs of each
- [ ] Multi-document transactions (yes, Mongo supports them — know when you'd actually reach for one)

### 6.3 The SQL vs. NoSQL Decision Framework

This is genuinely a common interview question ("when would you pick Mongo over Postgres?") — have a real answer, not a memorized one:

| Reach for SQL when... | Reach for NoSQL when... |
|---|---|
| Data has clear relationships you need to enforce (foreign keys) | Schema is flexible or evolves frequently per-record |
| You need multi-row transactional integrity (e.g., payments, inventory) | You're optimizing for very high write throughput over strict consistency |
| Your queries are relational (joins, aggregates across entities) | Your access pattern is mostly key-based lookups or document retrieval |
| Data integrity constraints matter more than raw write speed | Horizontal scaling of writes matters more than relational integrity |

Real systems often use **both** (polyglot persistence) — e.g., Postgres for core transactional entities, Mongo/Redis for logs, sessions, or denormalized read models. Being able to say *why* you'd combine them is a stronger answer than picking a side.

**Build this:** Design a normalized (3NF) Postgres schema for TeamForge's core entities (Organizations, Users, Teams, Projects, Tasks, Comments). Implement the access layer first in raw SQL, then migrate to Prisma with generated types. Add one real multi-statement transaction (e.g., "create project + seed default task list" must succeed or fail atomically). Add at least one composite index and show the `EXPLAIN ANALYZE` improvement before/after.

**Exit criteria:**
- [ ] Can explain normalization to 3NF from memory, with an example
- [ ] Can write a 3+ table JOIN query from scratch without a reference
- [ ] Can explain, out loud, why a specific operation in your project needed a transaction
- [ ] Can give a reasoned (not memorized) answer to "SQL or NoSQL for X scenario"

**Reference points:** PostgreSQL official docs/tutorial, Prisma docs, "SQL Murder Mystery" (fun, sharpens JOIN intuition fast).

---

## 7. Module 3 — Production-Grade API Development
**Weeks 8–12** (the largest module — this is where most junior interview questions live)

**Why it matters:** This is the module that separates "built a CRUD app" from "understands what production means." Interviewers probe this constantly: how do you handle auth failure, what happens when 1000 requests hit at once, how do you version a breaking change.

**Core concepts checklist**

*Auth & Authorization*
- [ ] JWT access + refresh token rotation, token revocation strategy
- [ ] RBAC (Owner/Admin/Member/Viewer) enforced via middleware, not scattered `if` checks

*Security*
- [ ] OWASP API Security Top 10 — quick reference below
- [ ] `helmet`, correctly scoped CORS, input sanitization, secrets management, `npm audit`

*Validation, Logging, Errors*
- [ ] Schema validation at the edge (Zod) applied consistently across routes
- [ ] Structured logging (pino or winston): log levels, request/correlation IDs, never logging secrets
- [ ] Centralized error-handling middleware, custom error classes, one consistent error response shape

*API Design*
- [ ] Versioning strategy (URI vs. header-based) — know the tradeoff, pick one, justify it
- [ ] Cursor-based pagination (preferred at scale) vs. offset pagination (know both, know why)
- [ ] Filtering & sorting query design that doesn't turn into an unmaintainable mess of query params

*Documentation & Production Conventions*
- [ ] OpenAPI/Swagger spec generated from (or alongside) code
- [ ] 12-factor config: env-based config with validation, health check endpoint, graceful shutdown

*Background Work*
- [ ] Queues with BullMQ + Redis: retries, backoff strategy, idempotency, scheduled/cron-style jobs

**OWASP API Security Top 10 — quick reference** *(check OWASP's site for the current revision, but these categories are the durable mental model)*

| # | Category | TeamForge example |
|---|---|---|
| 1 | Broken Object Level Authorization | Can User A fetch Task IDs belonging to a different Org? |
| 2 | Broken Authentication | Weak token handling, no refresh rotation |
| 3 | Broken Object Property Level Authorization | Mass assignment — can a Member set their own role to Admin? |
| 4 | Unrestricted Resource Consumption | No rate limiting on expensive endpoints |
| 5 | Broken Function Level Authorization | Can a Viewer hit an admin-only DELETE route directly? |
| 6 | Unrestricted Access to Sensitive Business Flows | No throttling on invite/signup flows |
| 7 | Server-Side Request Forgery | Fetching user-supplied URLs (e.g., avatar upload) unsafely |
| 8 | Security Misconfiguration | Default CORS `*`, verbose error stack traces in prod |
| 9 | Improper Inventory Management | Old, undocumented API versions still live |
| 10 | Unsafe Consumption of APIs | Trusting third-party API responses without validation |

**Build this:** Harden TeamForge: refresh-token rotation, RBAC middleware enforcing the Owner/Admin/Member/Viewer hierarchy, rate limiting, centralized error handling with custom error classes, cursor-based pagination + filtering on the tasks list, generated OpenAPI docs, and move notification emails + due-date digests into a BullMQ queue with retry/backoff.

**Exit criteria:**
- [ ] Can walk through 3+ OWASP API Top 10 items *as they apply to your own project*, not generically
- [ ] Can design a paginated, filterable endpoint from a blank file
- [ ] Can explain what makes your background job idempotent (what happens if it runs twice?)

**Reference points:** OWASP API Security Top 10, BullMQ docs, Zod docs, OpenAPI spec docs.

---

## 8. Module 4 — Testing Engineering
**Weeks 13–15**

**Why it matters:** "Do you write tests?" is asked in some form at almost every interview, and "how do you know your code works" is a near-universal follow-up. Untested code is also a red flag for team-readiness — it signals you'd be risky to review.

**Core concepts checklist**
- [ ] Jest fundamentals: structure, matchers, setup/teardown hooks
- [ ] Unit testing: pure functions and service-layer logic in isolation, mocking dependencies (`jest.mock`, manual mocks)
- [ ] Integration testing: hitting real routes with `supertest`, using a dedicated test database (or per-test transaction rollback)
- [ ] Mocking external services (email sending, third-party APIs)
- [ ] TDD thinking: red-green-refactor as a *tool you reach for on complex logic*, not a religion you apply to every trivial function
- [ ] Coverage: what it's useful for, and why chasing 100% is a waste of time

**Build this:** Unit-test TeamForge's business-logic/service layer (permission checks, task-status transitions). Integration-test at least 5 critical endpoints (auth flow, task CRUD, a permission-denied case) with `supertest` against a test database. Mock the email and queue dependencies so tests don't actually send emails. Wire up `npm test` with a coverage report.

**Exit criteria:**
- [ ] Can write a failing test first for a new feature, then implement to make it pass
- [ ] Can explain, with your own project as the example, when you'd write a unit test vs. an integration test

**Reference points:** Jest docs, Supertest docs.

---

## 9. Module 5 — Caching & Performance
**Weeks 16–17**

**Why it matters:** Caching questions ("how would you speed this up," "how do you invalidate a cache") show up constantly in system design and API design discussions, even at junior level.

**Core concepts checklist**
- [ ] Redis fundamentals: strings, hashes, lists, sets, sorted sets, TTLs
- [ ] Caching patterns: cache-aside (most common), write-through, write-behind
- [ ] Cache invalidation strategies: TTL-based, event-based (invalidate on write), versioned keys
- [ ] HTTP caching: `Cache-Control` directives, ETags, conditional requests (`If-None-Match`)
- [ ] Redis-backed rate limiting (sliding window / token bucket) — ties back to Module 3's security work

**Build this:** Cache TeamForge's expensive dashboard aggregation queries (tasks-per-status, overdue counts) in Redis with cache-aside + event-based invalidation on writes. Add ETag support to a key GET endpoint. Move your Module 3 rate limiter onto Redis for correctness across multiple app instances.

**Exit criteria:**
- [ ] Can explain the tradeoff between TTL-based and event-based invalidation with a concrete example from your own project
- [ ] Can explain why "cache invalidation is one of the two hard problems in computer science" — and mean it, not just quote it

**Reference points:** Redis docs, MDN's `Cache-Control` reference.

---

## 10. Module 6 — Deployment & DevOps
**Weeks 18–21**

**Why it matters:** Junior roles increasingly expect you to at least understand — if not own — how your code gets from a laptop to production. "Walk me through your CI/CD pipeline" is a very common interview prompt now.

**Core concepts checklist**
- [ ] Dockerfile for a Node app: multi-stage builds, image size optimization
- [ ] Docker Compose: orchestrating app + Postgres + Redis locally with one command
- [ ] CI fundamentals: automated linting + tests on every push/PR
- [ ] GitHub Actions: writing a workflow file, managing secrets in CI
- [ ] CD fundamentals: deploying to a live environment (a PaaS like Railway/Render, or a basic VPS)
- [ ] Environment management: per-environment `.env`, config vs. secrets, 12-factor principles
- [ ] Reverse proxies: Nginx basics, routing requests to your app
- [ ] HTTPS: TLS certs, Let's Encrypt/Certbot basics, why HTTPS is non-negotiable even for side projects
- [ ] Kubernetes — conceptual only 🔹: pods, deployments, services, why K8s exists at all. You do **not** need hands-on cluster experience for a junior role — you need to not freeze when it comes up in conversation.

**Build this:** Dockerize TeamForge (app + Postgres + Redis via Compose). Write a GitHub Actions pipeline that runs lint + the Module 4 test suite on every PR, and deploys to a live environment on merge to `main`. Serve it over HTTPS (via Nginx + Let's Encrypt if self-hosting, or managed HTTPS if using a PaaS).

**Exit criteria:**
- [ ] Can explain what every line in your Dockerfile and `docker-compose.yml` does
- [ ] Can walk someone through your CI/CD pipeline stage by stage
- [ ] Can explain, conceptually, what a Kubernetes "deployment" and "service" are — even without a cluster to point to

**Reference points:** Docker docs, GitHub Actions docs, Nginx docs, Certbot docs.

---

## 11. Module 7 — System Design Fundamentals + AI-Assisted Engineering
**Weeks 22–24**

**Why it matters:** Junior system design questions are common now, and they're not asking you to design Twitter — they want to see structured thinking about tradeoffs. Separately, being fluent with AI tools is quickly becoming an expected *skill*, not a shortcut — interviewers increasingly probe how you use them.

**Core concepts checklist**

*System Design*
- [ ] Scalability basics: vertical vs. horizontal scaling, statelessness
- [ ] Load balancing: round robin, least connections, health checks
- [ ] CAP theorem: consistency/availability/partition-tolerance tradeoffs — and where your own Postgres+Redis stack actually sits
- [ ] Message queues & event-driven architecture: pub/sub vs. queues, at-least-once vs. exactly-once delivery (ties directly back to your Module 3 BullMQ work)
- [ ] Monolith vs. microservices: the real tradeoffs, and *why a junior dev shouldn't default to microservices*
- [ ] Junior-level system design practice: back-of-envelope estimation, designing something like a URL shortener, a rate limiter, or a notification system at an appropriate depth

*AI-Assisted Backend Development*
- [ ] Using AI tools effectively for boilerplate, test generation, and doc generation — while you own the architecture decisions
- [ ] Reviewing AI-generated code critically, especially around auth logic and database queries (this is exactly where AI tools hallucinate confidently)
- [ ] Using AI to accelerate debugging (explaining unfamiliar stack traces/errors) without skipping the understanding step
- [ ] Being able to articulate, in an interview, *how* you use AI tools in your workflow — this is genuinely becoming a differentiator, not a liability, when framed as a deliberate practice

**Build this:** Write a 1-page system design doc for TeamForge as if scaling to 100k users — identify the actual bottlenecks (which queries, which endpoints) and propose concrete fixes using what you've already built (read replicas, more aggressive caching, offloading more work to queues). Practice 2–3 classic junior system design prompts on paper/whiteboard. Keep a short "AI usage log" of how you used AI tools across this roadmap — it's a genuinely good interview talking point.

**Exit criteria:**
- [ ] Can discuss CAP theorem tradeoffs concretely, referencing your own stack rather than abstractions
- [ ] Can sketch a system design answer for a common junior prompt in about 20 minutes
- [ ] Can describe one specific instance where you caught an AI tool's code suggestion being wrong or insecure

---

## 12. Module 8 — Capstone Polish + Interview Prep
**Weeks 25–26**

**Build this:**
- [ ] Polished README for TeamForge: architecture diagram, setup instructions, feature list
- [ ] Resume bullets, one per module (see mapping below) — concrete, quantifiable where possible
- [ ] Clean, pinned GitHub profile
- [ ] Mock interviews covering: DB design questions, API design questions, a live-debugging scenario, behavioral questions

**Resume bullet mapping (draft these, then tailor per JD):**

| Module | Draft resume bullet |
|---|---|
| 1 | "Built and migrated a Node.js backend to strict-mode TypeScript with runtime validation via Zod" |
| 2 | "Designed a normalized PostgreSQL schema with transactional integrity for multi-tenant data" |
| 3 | "Implemented RBAC, JWT refresh rotation, and OWASP-aligned security hardening on a production API" |
| 4 | "Built a Jest/Supertest test suite covering unit and integration paths for core business logic" |
| 5 | "Reduced dashboard query load using Redis caching with event-based invalidation" |
| 6 | "Containerized and deployed the application via Docker and a GitHub Actions CI/CD pipeline" |
| 7 | "Authored a system design proposal addressing scaling bottlenecks at 100k-user load" |

---

## 13. Guardrails Against Burnout & Confusion

Since you flagged this directly — here's the practical version, not the motivational-poster version:

- **One module at a time, no exceptions.** If you catch yourself reading about Kubernetes during Module 2, that's the signal to close the tab, not a signal you're being thorough.
- **Timebox rabbit holes to 2 days.** If a subtopic pulls you in beyond what's listed in a module's checklist, give yourself 2 days max to satisfy the curiosity, then move on. Come back to it later only if your project genuinely needs it.
- **"Done" = exit criteria met, not "I feel ready."** The feeling of mastery lags behind actual competence by weeks. Trust the checklist, not the feeling.
- **The checklist is the whole mountain, in advance.** You're not staring at an undefined pile of "everything to learn" — every single thing you need is already listed above, mapped to a week. There's nothing hiding outside this document.
- **A slipped date is not a failure.** The calendar dates are a compass, not a contract. If Module 3 takes 6 weeks instead of 5, that's normal — production API development usually does.

---

## 14. Master Topic Checklist

Every item you listed, mapped to where it's covered — so you can see, at a glance, that nothing is missing.

**TypeScript** (Module 1): production-grade TS ✅ · advanced typing ✅ · generics ✅ · utility types ✅ · type-safe Express ✅ · JS→TS migration ✅

**Database Engineering** (Module 2): MongoDB mastery ✅ · PostgreSQL mastery ✅ · SQL fundamentals ✅ · schema design ✅ · normalization ✅ · relationships ✅ · joins ✅ · indexing ✅ · transactions ✅ · query optimization ✅ · scaling fundamentals ✅ · SQL vs NoSQL ✅

**Production API Development** (Module 3): authN/authZ ✅ · security best practices ✅ · OWASP fundamentals ✅ · validation ✅ · logging ✅ · API versioning ✅ · pagination ✅ · filtering ✅ · error handling standards ✅ · performance optimization ✅ (+ Module 5) · documentation ✅ · background jobs ✅ · queues ✅ · production conventions ✅

**Testing** (Module 4): Jest ✅ · unit testing ✅ · integration testing ✅ · API testing ✅ · mocking ✅ · TDD thinking ✅

**Caching** (Module 5): Redis ✅ · server-side caching ✅ · cache invalidation ✅ · HTTP caching ✅ · Cache-Control ✅ · ETags ✅ · performance strategies ✅

**Deployment & DevOps** (Module 6): Docker ✅ · Docker Compose ✅ · CI/CD fundamentals ✅ · GitHub Actions ✅ · deployment pipelines ✅ · environment management ✅ · reverse proxies ✅ · HTTPS ✅ · basic K8s concepts ✅

**Engineering Fundamentals** (Module 7): system design basics ✅ · scalability ✅ · load balancing ✅ · CAP theorem ✅ · message queues ✅ · event-driven architecture ✅ · monolith vs. microservices ✅ · AI-assisted backend development ✅ · effective AI tool use ✅

---

*Start date: Module 1, Week 1 — whenever you say go.*
