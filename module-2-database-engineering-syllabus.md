# Module 2 — Database Engineering
### A chapter-wise, background-calibrated syllabus for TeamForge

This is the detailed syllabus for Module 2 of your roadmap, scoped specifically to what you *don't* already know. It assumes the Roadmap and TeamForge Blueprint documents as context.

---

## How to Read This Document

Every chapter is tagged with a focus level based on your stated background (Mongoose/MongoDB: schema modeling, Compass, RBAC modeling, TTL jobs, sessions, transactions, basic CRUD, light aggregation — and foundational MySQL: tables, joins, normalization):

| Tag | Meaning |
|---|---|
| **SKIP** | You already have this. Mentioned only where it's a prerequisite for something new. |
| **LIGHT** | Fast refresher — mostly new tooling/syntax, not new concepts. |
| **MEDIUM** | You know the concept from Mongo; the relational-specific mechanics are new. |
| **DEEP** | Genuine gap. Spend real time here. |

**Two architectural decisions baked into this syllabus, so you know why things are split the way they are:**
1. **PostgreSQL + Prisma is your primary data layer** for everything relational (the Org→Team→Project→Task hierarchy) — this is where your real gap is, and where the syllabus spends most of its time.
2. **MongoDB stays in the project deliberately**, for `ActivityLog` and `Notification` — not as a separate exercise, but because those two entities are genuinely append-only and schema-flexible, which is exactly when NoSQL is the right call. This turns your existing Mongo skill into a real, defensible architectural decision instead of a thing you learned once and dropped.

---

## At a Glance

| Ch | Title | Focus | Days | TeamForge Deliverable |
|---|---|---|---|---|
| 1 | Setup + Postgres-from-MySQL Bridge | LIGHT | 1–2 | Local Postgres running, first raw table |
| 2 | Schema Design & DDL for TeamForge | LIGHT theory / DEEP practice | 3 | Full raw-SQL schema for all MVP entities |
| 3 | Indexing Deep Dive | DEEP | 3 | Indexes + before/after `EXPLAIN ANALYZE` |
| 4 | Transactions & Data Integrity in Postgres | MEDIUM | 2 | 2 real atomic transactions |
| 5 | Advanced Querying — CTEs & Window Functions | MEDIUM–DEEP | 3 | Dashboard + activity queries |
| 6 | Query Optimization & Connection Pooling | DEEP | 2 | N+1 fix + `pg.Pool` setup |
| 7 | Prisma — Type-Safe Data Layer | DEEP | 4 | Full `schema.prisma`, migrated |
| 8 | Scaling Fundamentals + Applied SQL vs NoSQL | LIGHT / MEDIUM | 2 | Written decision doc |
| 9 | MongoDB Deepening — ActivityLog & Notification | LIGHT–MEDIUM | 3 | Aggregation feed + TTL cleanup |
| 10 | Capstone Integration + Exit Criteria | — | 2 | End-to-end polyglot proof |

**Total: ~26 days**, inside your 4-week Module 2 window with buffer to spare.

---

## Chapter 1 — Setup + Postgres-from-MySQL Bridge
**Focus: LIGHT · 1–2 days**

**Skip:** basic `SELECT`/`WHERE`/`ORDER BY`, basic `INNER`/`LEFT JOIN` syntax — identical enough to MySQL that there's nothing new to learn here.

**What's actually new:**
- Postgres tooling: `psql` (CLI) + a GUI client (pgAdmin or TablePlus — same role Compass plays for Mongo)
- Fastest local setup: `docker run --name teamforge-pg -e POSTGRES_PASSWORD=devpassword -p 5432:5432 -d postgres:16` — one command, no install. You'll learn what's actually happening here properly in Module 6; for now, just treat it as "a Postgres server is running." A native install works just as well if you prefer it.
- Dialect differences that matter day-to-day: double-quoted identifiers (not backticks), `GENERATED ALWAYS AS IDENTITY` (modern replacement for `AUTO_INCREMENT`) vs. `gen_random_uuid()` for UUID primary keys, the `RETURNING` clause (Postgres gives you the inserted/updated row back — MySQL makes you re-query for it), `ON CONFLICT (...) DO UPDATE/NOTHING` (Postgres's upsert, vs. MySQL's `ON DUPLICATE KEY UPDATE`), and native rich types: `JSONB`, `ARRAY`, `ENUM`.

**Build this:** Get Postgres running, connect via `psql`, create the `teamforge_dev` database. Write one raw `CREATE TABLE users (...)` using `GENERATED ALWAYS AS IDENTITY`, insert a row with `RETURNING *`, and actually look at what comes back — this pattern (insert-and-get-the-row-back) is one you'll use constantly once you're back in your API layer.

**Checkpoint:** You can connect via `psql` unassisted, and you can explain in one sentence why `RETURNING` saves a round-trip that MySQL requires.

---

## Chapter 2 — Schema Design & DDL for TeamForge
**Focus: LIGHT theory / DEEP practice · 3 days**

**Skip:** re-deriving what 1NF/2NF/3NF mean — one-line refresher only: *1NF = atomic columns, no repeating groups. 2NF = no partial dependency on part of a composite key. 3NF = no transitive dependency on a non-key column.* You know this — the new work here is applying it across a real 9-entity domain, not a 2-table textbook example.

**What's new:** Postgres foreign key syntax, and — more importantly — deliberately choosing an `ON DELETE` behavior for every single relationship instead of leaving it as an afterthought.

**Build this:** Write complete `CREATE TABLE` statements for every MVP entity from the Blueprint (`users`, `organizations`, `org_memberships`, `teams`, `team_memberships`, `projects`, `tasks`, `comments`, `invitations`) as raw SQL in a `schema.sql` file. For every foreign key, explicitly choose and comment the `ON DELETE` behavior — in particular:
- `tasks.assignee_id` → `ON DELETE SET NULL` (this is Workflow G from the Blueprint — a removed member's tasks become unassigned, not deleted)
- `org_memberships.user_id` / `org_id` → `ON DELETE CASCADE` (a membership row is meaningless without both sides)
- `tasks.project_id`, `projects.team_id`, `teams.org_id` → `ON DELETE CASCADE` (matches the Blueprint's strict hierarchy — nothing outlives its parent)

**Checkpoint:** `psql -f schema.sql` runs clean with no errors, and you can justify — entity by entity, out loud — why each FK got the `ON DELETE` behavior it did.

---

## Chapter 3 — Indexing Deep Dive
**Focus: DEEP · 3 days**

This is a genuine gap and worth the full 3 days.

**What's new:** B-tree indexing (conceptually — how it turns a scan into a lookup), composite index column ordering (the "leftmost prefix" rule — an index on `(a, b)` helps queries filtering on `a` alone or `a AND b`, but not `b` alone), unique indexes vs. unique constraints, partial indexes, and reading `EXPLAIN ANALYZE` output well enough to tell a sequential scan from an index scan and understand the cost estimates.

**A real design tension you'll hit here (this is deliberate, not a mistake in the Blueprint):** `tasks` doesn't have a direct `org_id` — it's only reachable via `project → team → org`. But your #1 business rule is *tenant isolation must be absolute and fast*. This is where you make a real call: either always join through the hierarchy to scope every tasks query by org, or denormalize an `org_id` column directly onto `tasks` purely for query-speed and indexing — accepting that it's not strictly 3NF anymore. This exact tension shows up constantly in real multi-tenant systems. Make the call, and once you have, update the Blueprint's Task entity to reflect it — the Blueprint should always match what you actually built.

**Build this:**
- `UNIQUE` index on `organizations.slug` and `users.email` (enforces the Blueprint's uniqueness rules at the DB level, not just in application code)
- Resolve the `org_id`-on-`tasks` tension above, then add a composite index — `(org_id, status)` if you denormalized, or an index supporting your join path otherwise
- Composite index on `(project_id, status)` for the dashboard query
- Run `EXPLAIN ANALYZE` on a tasks-by-org query before and after indexing, and keep the before/after output side by side

**Checkpoint:** You can read an `EXPLAIN ANALYZE` plan and immediately say whether it used a sequential scan or an index scan, and you can defend your denormalization call (or your decision not to denormalize) with a specific reason.

---

## Chapter 4 — Transactions & Data Integrity in Postgres
**Focus: MEDIUM · 2 days**

**Skip:** re-explaining what atomicity/consistency/isolation/durability mean in the abstract — you already have this from Mongo transactions.

**What's new:** `BEGIN` / `COMMIT` / `ROLLBACK` syntax in raw SQL, Postgres's default isolation level (`READ COMMITTED`) and knowing `SERIALIZABLE` exists for stricter guarantees, what a deadlock actually is and how Postgres detects and aborts one side of it, and the practical difference between a Mongo multi-document transaction and a Postgres one — same ACID contract, different locking model underneath.

**Build this:** Implement two real transactions in raw SQL:
1. **Workflow G** — removing an org member: reassign their tasks (`UPDATE tasks SET assignee_id = NULL WHERE assignee_id = $1`) and delete their membership row, both inside one transaction.
2. **Project creation with a seed task** — insert into `projects`, then insert a default "Getting Started" task referencing the new project's id, atomically. If the second insert fails, the project shouldn't exist either.

**Checkpoint:** For both transactions, walk through out loud what happens if the *second* statement fails — what gets rolled back, and why that matters.

---

## Chapter 5 — Advanced Querying: CTEs & Window Functions
**Focus: MEDIUM–DEEP · 3 days**

**Skip:** basic single-table joins — go straight to multi-table.

**What's new:** `WITH` (CTEs) for breaking a complex query into readable steps, and window functions (`ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)`, `RANK()`) for "latest/top N per group" queries — a pattern you'll hit constantly and can't do cleanly with `GROUP BY` alone.

**Build this:** Write these three queries for real:
- All tasks in an org with project name, team name, and comment count (3-table join + aggregate)
- Most recent comment per task, using `ROW_NUMBER() OVER (PARTITION BY task_id ORDER BY created_at DESC)`
- **Workflow F**, the dashboard query: task count grouped by status per project, plus an overdue count, as a single query using a CTE to stage the intermediate grouping

**Checkpoint:** You can explain, with your own queries as examples, when a CTE is purely for readability vs. when a window function is doing something a plain `GROUP BY` genuinely can't.

---

## Chapter 6 — Query Optimization & Connection Pooling
**Focus: DEEP · 2 days**

**What's new:** the N+1 query problem, concretely — not as a term you can define, but as a bug you've caused and fixed yourself. Also: connection pooling (`pg.Pool`) and why a production app never opens a fresh connection per request.

**Build this:** Deliberately write the N+1 version first — fetch N tasks, then loop and issue a separate query for each task's comments. Watch the query count. Then rewrite it as one batched query (`WHERE task_id = ANY($1)` or a proper join). Set up `pg.Pool` with sane defaults (max connections, idle timeout) in your TS app's DB config — this becomes the actual connection layer TeamForge uses from here on.

**Checkpoint:** You can explain the N+1 problem to someone else using your own tasks/comments example, not a generic textbook one.

---

## Chapter 7 — Prisma: Type-Safe Data Layer
**Focus: DEEP · 4 days**

**What's new:** `schema.prisma` modeling, `prisma migrate dev`, the generated type-safe client, relation queries (`include`/`select`), the `$transaction` API, and — the part that actually matters — seeing the SQL Prisma generates under the hood (via query logging or `prisma studio`) and comparing it to what you hand-wrote in earlier chapters. Raw SQL first, then Prisma, so Prisma stays a tool you understand rather than a black box you trust.

**Build this:**
- Model the full schema in `schema.prisma`, mirroring Chapter 2 exactly — same tables, same relations, same `onDelete` behaviors expressed in Prisma's syntax
- Run your first migration, confirm it matches your hand-written schema
- Re-implement the Chapter 4 Workflow G transaction using `prisma.$transaction([...])`, and compare its generated SQL to your raw version
- Re-implement the Chapter 5 dashboard aggregation using Prisma's query API where it stays clean — and explicitly find the point where it gets awkward enough that you'd drop to `prisma.$queryRaw` instead. That judgment call is the actual skill here, not memorizing Prisma's API.

**Checkpoint:** You can name one specific query from your own project where you'd choose raw SQL/`$queryRaw` over Prisma's query builder, and say why.

---

## Chapter 8 — Scaling Fundamentals + Applied SQL vs. NoSQL
**Focus: LIGHT conceptual / MEDIUM applied · 2 days**

**Conceptual (junior-interview depth only — you need to discuss these, not implement them):** read replicas (writes to primary, reads from replicas), vertical vs. horizontal scaling, when sharding becomes a real conversation (at a scale TeamForge will never reach — know that it exists and roughly why).

**Applied decision (this is the real work of this chapter):** Formally justify, in writing, why `ActivityLog` and `Notification` live in MongoDB while everything else lives in Postgres — using your own project's actual write patterns as the argument, not the generic decision framework from the Blueprint.

**Build this:** A half-page decision doc, written as if defending it in an interview: why Postgres for the core hierarchy, why Mongo specifically for these two collections.

**Checkpoint:** You can defend this decision out loud, using TeamForge specifics, in under 2 minutes.

---

## Chapter 9 — MongoDB Deepening: ActivityLog & Notification
**Focus: LIGHT–MEDIUM · 3 days**

**Skip:** Mongoose basics, CRUD, schema modeling fundamentals, sessions, basic transactions — you already have all of this solidly.

**What's new:**
- Aggregation pipeline stages beyond what you've used so far (`$group`, `$sort`, `$limit` chained together for a real feed query)
- A cross-database modeling decision: Mongo can't enforce a real foreign key into Postgres, so referencing a Postgres user's id in an `ActivityLog` document means that link is app-level only, not DB-enforced. Given that, you'll **embed** the actor's display name directly into the log entry at write time (a deliberate denormalization) rather than trying to "join" back to Postgres on every read.
- Applying a TTL index in a *new* context: not fixed-expiry, but conditional — expiring 90 days after `read_at` is set, not from `created_at`.

**Build this:**
- `ActivityLog` and `Notification` Mongoose models, referencing Postgres entity ids as plain UUID strings
- An aggregation pipeline: activity feed for an org over the last 7 days, grouped by `action_type`, most recent first
- Compound index on `(org_id, created_at)` to support that feed query
- TTL index on `Notification` that auto-deletes a notification 90 days after `read_at` is set

**Checkpoint:** You can explain why the actor's name is embedded rather than referenced in `ActivityLog`, and what it costs you if that user later changes their display name (stale historical names in old log entries — and why that's actually the *correct* tradeoff for an audit log).

---

## Chapter 10 — Capstone Integration + Module 2 Exit Criteria
**Focus: Integration · 2 days**

**Build this:** Wire it together. Your TS service layer now talks to Postgres (via Prisma, dropping to raw SQL where Chapter 7 decided that made sense) *and* MongoDB (via Mongoose) in the same application. Write one end-to-end script that creates an org + task in Postgres, logs the action to MongoDB's `ActivityLog`, then reads both back — proof the polyglot setup actually works together, not just in isolation.

### Module 2 Exit Criteria

- [ ] Can explain normalization to 3NF using your own TeamForge tables as the example
- [ ] Can write a 3+ table JOIN from scratch, no reference material
- [ ] Can justify every `ON DELETE` choice in your schema, entity by entity
- [ ] Can read an `EXPLAIN ANALYZE` plan and identify index scan vs. sequential scan
- [ ] Can defend your `org_id`-on-`tasks` denormalization call (or your decision not to make it)
- [ ] Can name a specific case where you chose raw SQL/`$queryRaw` over Prisma
- [ ] Can defend the Postgres-vs-Mongo split for TeamForge's data using your own project, not the generic framework
- [ ] Can explain the embedding decision on `ActivityLog` and its tradeoff

When every box is checked, you're done with Module 2 — move to Module 3 (Production-Grade API Development) from the master roadmap.
