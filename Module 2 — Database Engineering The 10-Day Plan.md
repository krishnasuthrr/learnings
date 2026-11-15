# Module 2 — Database Engineering: The 10-Day Plan

*From first principles to hire-ready: PostgreSQL + MongoDB, applied to TeamForge. This replaces the earlier chapter-wise syllabus: same destination, new route.*

## The Deal (read once)

**What 'mastery' means here.** The depth a hiring manager actually tests at junior level: you can model data, write and debug real queries, explain why a query is slow and fix it, reason about transactions and concurrency, and pick a database and defend the choice. DBA-level internals are not the goal; they sit in the Deferred list at the end.

**Pace.** About 5 focused hours a day for 10 days (\~50 hrs). You have touched SQL and Mongo before, so Days 2, 3 and 8 move fast, but nothing is assumed and Day 1 starts from first principles. At 2–3 hrs/day, stretch to about 3 weeks instead of cutting content: the exit criteria are the contract, not the day count.

**Daily rhythm (identical every day, so you never wonder what to do):**

| Block | Time | What |
| --- | --- | --- |
| Learn | \~1.5 h | Only the listed topics |
| Mini project | \~2 h | The concept-clicker. Do it before touching TeamForge |
| TeamForge step | \~1 h | Apply the same idea to the real project |
| Recap | \~30 min | Write 5 lines starting 'I now understand why…' |

**Three rules**

1. **Break it on purpose.** Every mini project makes you hit the failure first, then fix it. Don't skip to the fix.
2. **Type every statement.** No copy-paste from tutorials.
3. **If it's not in this plan, don't chase it.** A rabbit hole longer than 30 minutes goes into a 'later' note and you move on.

## The 10-Day Map

| Day | Theme | Mini project | TeamForge deliverable |
| --- | --- | --- | --- |
| 1 | What a database is | Build a database from scratch | Postgres running |
| 2 | SQL core | Constraint torture + query kata | users, organizations tables |
| 3 | Relational design | Rescue the spreadsheet | Full schema.sql |
| 4 | Joins, CTEs, windows | SQL gym on 100k rows | seed.sql + dashboard query |
| 5 | Indexes and query plans | Slow-query lab (1M rows) | Indexes + performance notes |
| 6 | Transactions, concurrency | Race-condition lab | 3 atomic transactions |
| 7 | Node + Prisma | Same feature, three ways | schema.prisma + repositories |
| 8 | MongoDB | Event warehouse | ActivityLog + Notification |
| 9 | Choosing, scaling, operating | Pick-the-database + disaster drill | Decision record |
| 10 | Integration + exit | End-to-end script + oral exam | Exit checklist |

## Day 1 — What a database actually is

**Learn**

- Why not just files? The six problems a database exists to solve: fast search, concurrent access, crash safety, integrity rules, access control, scale
- Anatomy of a DBMS: client → parser → planner → executor → storage engine → disk, with a memory buffer between. Rows live in fixed-size pages; reading a page is the unit of cost
- Why disk access dominates, and how an index (a B-tree) turns a scan into a lookup (concept only; Day 5 goes deep)
- Write-ahead log in one idea: record the change in a log first so a crash can be replayed. This is the root of durability
- Relational model: tables, rows, columns, keys, constraints, schema. SQL is declarative: you say what, the planner decides how
- Data model families in one line each (relational, document, key-value, wide-column, graph); OLTP vs OLAP (we only do OLTP)
- Client–server over TCP (Postgres 5432, Mongo 27017), roles and authentication

**Mini project — Build a database from scratch (\~2 h, plain TypeScript, no libraries)** A key-value store in one file: set(key, value), get(key), delete(key). Each write appends a line to a log file; an in-memory map holds the current values; on restart the log is replayed to rebuild the map. Then break it on purpose and write down what happens:

1. Kill the process mid-write: is the file corrupted?
2. Run two processes writing at once: what is lost?
3. 'Find every value containing x': how does it scale?
4. Store 1M keys and restart: how long does startup take?

For each failure, name the database feature that solves it (WAL and checksums, locking and transactions, indexes, checkpointing). That mapping is the whole point.

**TeamForge step.** Run Postgres with `docker run --name teamforge-pg -e POSTGRES_PASSWORD=devpass -p 5432:5432 -d postgres` (Docker is studied properly in Module 6; for now it just means a Postgres server is running). Connect with psql plus a GUI (pgAdmin or TablePlus), learn `\l`, `\c`, `\dt`, `\d`, `\x`, `\timing`, and create `teamforge_dev`.

**Done when** you can explain, unaided, what happens between typing a SELECT and getting rows back, and which database feature fixes each of your four failures.

## Day 2 — SQL core

**Learn**

- Postgres types that matter: integer/bigint, numeric (never float for money), text, boolean, timestamptz (always, not timestamp), date, uuid, jsonb, enum, arrays. Identity columns vs UUID primary keys
- DDL: CREATE / ALTER / DROP, schemas, IF NOT EXISTS
- Constraints: PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL, CHECK, DEFAULT. Mindset: constraints are the database refusing bad data so no application bug can corrupt it
- DML: INSERT (multi-row), UPDATE, DELETE, RETURNING, upsert with ON CONFLICT
- SELECT: WHERE, IN, BETWEEN, LIKE/ILIKE, ORDER BY, LIMIT/OFFSET, DISTINCT, CASE
- NULL: three-valued logic, IS NULL, COALESCE, why `= NULL` never matches
- Aggregates and GROUP BY / HAVING. WHERE filters rows, HAVING filters groups. Logical order: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT

**Mini project — Constraint torture + query kata** Part A: create `authors`, `books`, `members`, `loans` using every constraint type. Then violate each one on purpose (duplicate key, FK to a missing row, NULL in NOT NULL, failing CHECK, deleting a parent that has children) and keep a table of the exact error messages. Part B: seed \~40 rows and answer with single statements: books per author; authors with no books; members with overdue loans; the most-borrowed book; average loan length per member; and one count written both with WHERE and with HAVING, explaining the difference.

**TeamForge step.** Create `users` and `organizations` with real constraints (unique email, unique slug with a CHECK on its format, `created_at timestamptz DEFAULT now()`). Insert with RETURNING and test an upsert on slug.

**Done when** you write CRUD plus GROUP BY / HAVING from a blank file without lookups, and can recite the execution order.

## Day 3 — Relational design and normalization

**Learn**

- ER modelling: entities, relationships, cardinality (1:1, 1:N, M:N). A 1:N puts the FK on the many side; an M:N needs a join table, which can carry its own data (like `role` on org\_memberships)
- Keys: primary, candidate, composite, surrogate vs natural. Identity vs UUID tradeoffs (index locality, guessability, generating IDs outside the DB)
- Referential actions: ON DELETE CASCADE / SET NULL / RESTRICT. Decide by asking: can the child exist without the parent?
- Normalization: every non-key column depends on the key, the whole key, and nothing but the key (1NF atomic values, 2NF no partial dependency, 3NF no transitive dependency). Why it matters: insert, update and delete anomalies
- Deliberate denormalization and when it's correct (price snapshot at purchase, tenant id on every table)
- Patterns: soft delete, audit columns, status enum vs lookup table, multi-tenancy via tenant id

**Mini project — Rescue the spreadsheet** Create one flat table `orders_flat(order_id, customer_name, customer_email, customer_city, products, prices, order_date, shipping_address)` where `products` and `prices` hold comma-separated values, with \~15 rows including one customer spelled two ways.

1. Prove each anomaly by experiment: change one customer's city (how many rows?), delete a customer's only order (what is lost?), try to add a customer with no order
2. Decompose to 3NF: customers, products, orders, order\_items
3. Write the DDL and migrate the data with INSERT … SELECT
4. Rebuild the original flat shape as a `CREATE VIEW` using joins
5. Name one deliberate denormalization (`price_at_purchase` on order\_items) and explain why it is correct

**TeamForge step.** Write the full `schema.sql` for the Blueprint's MVP entities: users, organizations, org\_memberships, teams, team\_memberships, projects, tasks, comments, invitations. Put a one-line justification on every foreign key (assignee\_id → SET NULL; memberships and the Org → Team → Project → Task chain → CASCADE). Decide whether tasks carries a denormalized org\_id, record the decision, and update the Blueprint's Task entity to match.

**Done when** you can name the normal form any table violates and why, and every FK in schema.sql has a justification.

## Day 4 — Joins, subqueries, CTEs, window functions

**Learn**

- Joins: INNER, LEFT, FULL, CROSS, self-join. The trap: a filter on a LEFT JOIN in WHERE vs in ON. Mental model: 1:N joins multiply rows
- Subqueries: scalar, IN vs EXISTS, correlated
- CTEs (WITH) for readable multi-step queries; know recursive CTEs exist (org and team trees)
- Window functions: ROW\_NUMBER, RANK, SUM() OVER, LAG, with PARTITION BY and ORDER BY
- UNION / UNION ALL / INTERSECT / EXCEPT; views vs materialized views
- JSONB basics: `->`, `->>`, `@>`

**Mini project — SQL gym on 100k rows** Seed with `generate_series` and `random()`: 5 orgs, 50 teams, 500 projects, 100k tasks, 300k comments. Answer the ten questions below, and before running each, write down the row count you predict. That forces you to understand join multiplication.

1. Tasks with project, team and org names
2. Projects with zero tasks: via LEFT JOIN … IS NULL, then via NOT EXISTS
3. Latest comment per task (window function)
4. Top 3 assignees per org by completed tasks
5. Overdue tasks per team
6. The same LEFT JOIN with the filter in WHERE vs in ON, explaining the different results
7. Tasks created per day with a running total
8. Tasks with no comments
9. Users who belong to more than one org
10. Status counts per project as columns (CASE or FILTER)

**TeamForge step.** Keep the seed as `seed.sql`. Write the Workflow F dashboard query (task counts by status per project plus an overdue count) in `queries/dashboard.sql`.

**Done when** you predict the row count correctly on 8 of 10 before running, and can explain WHERE vs ON without notes.

## Day 5 — Indexes and query plans

**Learn**

- How a B-tree index works: sorted, balanced, page-based, so lookups are logarithmic and range scans are cheap. Scan types: seq, index, index-only, bitmap
- The planner: statistics and cost estimates, and why it sometimes ignores your index (low selectivity, small table, stale stats → ANALYZE)
- Composite indexes and the leftmost-prefix rule (equality columns first, range last); covering indexes (INCLUDE); partial, unique and expression indexes (`lower(email)`); GIN for jsonb and arrays (concept)
- Index costs: slower writes, more storage. Postgres does not auto-index foreign keys; you must
- EXPLAIN vs `EXPLAIN (ANALYZE, BUFFERS)`: node types, estimated vs actual rows, loops
- OFFSET pagination slows down the deeper you go; keyset (cursor) pagination does not. Module 3 will use cursors
- The N+1 query problem (concept now; you will cause it in code on Day 7)

**Mini project — Slow-query lab (1M rows)** Scale the seed to 1M tasks. Take five slow queries: filter by org and status; join through project to team; `ORDER BY created_at DESC LIMIT 20`; lookup by `lower(email)`; `OFFSET 500000`. For each: baseline EXPLAIN ANALYZE (time and scan type) → hypothesis → one index → re-measure. Keep a results table (query | before ms | index | after ms | scan type). Then sabotage on purpose: (a) a composite index with the wrong column order, (b) a function on an indexed column without an expression index, (c) an index on a low-selectivity boolean. Watch the planner ignore each and explain why. Finish by comparing OFFSET vs keyset on a deep page.

**TeamForge step.** Apply the winning indexes to schema.sql, index every foreign key, add unique indexes for email and slug, and keep the results table in `docs/db-performance.md`.

**Done when**, shown any EXPLAIN ANALYZE output, you can name the scan type, spot an estimate-vs-actual mismatch, and say which index you would try.

## Day 6 — Transactions and concurrency

**Learn**

- ACID in plain words; BEGIN / COMMIT / ROLLBACK / SAVEPOINT; autocommit
- Anomalies: dirty read, non-repeatable read, phantom read, lost update, write skew. Postgres levels: READ COMMITTED (default), REPEATABLE READ, SERIALIZABLE (retry on serialization failure). MVCC in one idea: readers don't block writers, and each transaction sees a snapshot
- Locks: row locks, `SELECT … FOR UPDATE` (and SKIP LOCKED, a preview of job queues in Module 3), advisory locks (awareness). Deadlocks: caused by opposite lock order, resolved by Postgres aborting one side, prevented by consistent ordering
- Safe-write patterns: atomic `UPDATE … SET n = n - 1 WHERE n > 0`; UNIQUE as an idempotency guard; optimistic locking with a version column
- Contrast with Mongo: single-document atomicity vs multi-document transactions

**Mini project — Race-condition lab (two psql sessions first, then Node scripts)** Scenario: only one user may claim a task, and a counter must never go negative.

1. Reproduce a lost update with read-then-write under READ COMMITTED
2. Fix it three ways (atomic UPDATE, SELECT FOR UPDATE, optimistic version column) and note each tradeoff
3. Reproduce a deadlock with two transactions locking two rows in opposite order, then fix it by ordering
4. Show a non-repeatable read under READ COMMITTED, then show it disappear under REPEATABLE READ
5. Fire 50 concurrent claim requests from Node with Promise.all and prove exactly one wins

**TeamForge step.** Implement three transactions in raw SQL: (a) Workflow G: remove a member = null their task assignments + delete their membership; (b) create a project + seed a 'Getting Started' task; (c) accept an invitation = create membership + mark invitation accepted, idempotent via a UNIQUE constraint.

**Done when** you can narrate a lost update step by step and name three ways to prevent it.

## Day 7 — Postgres from Node, then Prisma

**Learn**

- Driver layer (`pg`): Pool vs Client, parameterized queries ($1, $2) and why they stop SQL injection, running a transaction on one checked-out client and releasing it, pool sizing (one pool per process, stay under the server's max\_connections)
- Migrations: schema changes as versioned, forward-only files; expand/contract for safe changes (concept)
- ORM tradeoffs: you gain types, productivity and relations; you pay with hidden SQL, N+1 and awkward aggregates
- Prisma: `schema.prisma` (models, relations, enums, @@index, @@unique, @map), `migrate dev` vs `migrate deploy`, `db pull`, CRUD, `include` / `select`, cursor pagination, `$transaction` (array and interactive), `$queryRaw` (tagged templates are parameterized), query logging, error codes P2002 (unique violation) and P2025 (record not found), one client instance in TS

**Mini project — Same feature, three ways** Feature: list an org's open tasks with project name and comment count, paginated. Build it (A) with raw `pg`, (B) with Prisma's query API, (C) with Prisma `$queryRaw`. Compare lines of code, the generated SQL (turn on query logging), type safety, and EXPLAIN results. Then cause an N+1 in Prisma on purpose (a findMany per task in a loop) and fix it with `include` or groupBy. Finally trigger a P2002 and map it to a clean error object your API can return. Close with a 5-line verdict: when raw, when Prisma.

**TeamForge step.** Write `schema.prisma` mirroring schema.sql (use `db pull` as a cross-check), baseline the initial migration, build repository functions for users, orgs, memberships and tasks, and re-implement Workflow G with `$transaction`.

**Done when** you can show the SQL behind any Prisma call and defend one place where you chose `$queryRaw`.

## Day 8 — MongoDB, first principles to working depth

You have used Mongo, so basics go fast; the time goes to modelling, indexing and aggregation depth.

**Learn**

- Document model and BSON; ObjectId; flexible schema is not no schema (validation via $jsonSchema or Mongoose). What Mongo optimizes: aggregate-shaped reads, flexible shapes, horizontal scale
- Modelling: embed when data is read together, bounded, and rarely updated on its own; reference when it is unbounded or shared. Patterns: extended reference (copy a name), subset, bucket, pre-computed, schema versioning. Anti-patterns: unbounded arrays, `$lookup` everywhere (a relational design in the wrong database)
- Indexes: single, compound (ESR rule: Equality, Sort, Range), multikey, TTL, partial, unique, text. `explain('executionStats')`: COLLSCAN vs IXSCAN, docsExamined vs nReturned
- Aggregation: $match (early), $project / $addFields, $group, $sort, $limit, $unwind, $lookup, $facet. Why $match and $sort first lets indexes work
- Durability: write concern and read concern basics, replica sets and elections (concept), the cost of multi-document transactions, change streams (awareness)

**Mini project — Event warehouse** Seed 200k `activity_logs` documents (org\_id, actor, action\_type, target, metadata whose shape varies by action\_type, created\_at).

- **Part A, eight aggregation puzzles:** events per day for the last 7 days; top 5 actors per org; distribution by action\_type; busiest hour of day; latest event per actor; unique actors per week; a per-task funnel (created → assigned → done); events per org with the actor's name via `$lookup`, then redone with a denormalized name
- **Part B, index lab:** run the feed query (org, last 7 days, newest first), observe COLLSCAN, add a compound index following ESR and compare docsExamined, then reorder the keys wrongly and watch it degrade
- **Part C, TTL:** a notifications collection that auto-deletes 90 days after `read_at` is set; verify with a short test TTL (cleanup runs about once a minute)

**TeamForge step.** Mongoose models for ActivityLog and Notification (Postgres IDs stored as plain strings: integrity is app-level, since no database can enforce a cross-database key), the activity-feed aggregation, a compound index on (org\_id, created\_at), TTL on notifications, and the actor's display name embedded at write time.

**Done when**, given a new scenario, you can choose embed vs reference in 60 seconds with a reason, and read any explain output.

## Day 9 — Choosing, scaling, operating

**Learn**

- SQL vs NoSQL decision framework: access patterns, relationships, integrity needs, schema volatility, scale shape, team skills. The cost of polyglot persistence: two systems to run, no cross-database transactions
- CAP and consistency in practice: strong vs eventual, and why replication lag matters (reading your own write from a replica)
- The scaling ladder, cheapest first: fix queries and indexes → cache → bigger machine → read replicas → partitioning → sharding. Postgres: pooling with PgBouncer, partitioning and streaming replication (awareness). Mongo: replica sets, sharding and shard-key choice (high cardinality, matches your queries)
- Operating a database: backups (`pg_dump` / `pg_restore`, logical vs physical, point-in-time recovery as awareness), production migrations (never edit an applied migration), least-privilege roles (the app user is not a superuser), secrets in env vars, slow-query visibility (`pg_stat_statements`, Mongo profiler)

**Mini project — Pick-the-database drill + disaster drill** Written, \~90 min. For four scenarios (e-commerce orders and inventory; chat message history; IoT sensor metrics; a CMS with variable product attributes) write: the chosen database(s), a schema or collection sketch, the two hottest queries, the index plan, what breaks at 100× scale, and one tradeoff you knowingly accept. Disaster drill: `pg_dump` TeamForge, drop the database, restore, verify the row counts match. Do the equivalent with mongodump / mongorestore.

**TeamForge step.** Write a one-page decision record: why Postgres for the core hierarchy, why Mongo for ActivityLog and Notification, what you accept (no cross-database foreign keys or transactions, denormalized actor names), and what you would change at 100× scale. Create a least-privilege `teamforge_app` role and point the app at it.

**Done when**, given a new scenario, you can pick and defend a database in two minutes and say what fails first at scale.

## Day 10 — Integration and exit

**Build.** One end-to-end script or integration test: create user + org + membership (Postgres transaction) → create a project and task → write an ActivityLog entry to Mongo → run the dashboard query (Postgres) and the activity feed (Mongo) → remove the member (transaction) → verify the tasks are unassigned and the log entry exists → EXPLAIN the dashboard query to confirm it uses an index.

**Oral exam.** Answer these out loud or on a whiteboard, no notes. Any you stumble on: redo that day's mini project, not the reading.

1. What happens between typing a SELECT and getting rows back?
2. Normalize this table to 3NF, then argue for one denormalization
3. Why does a LEFT JOIN filtered in WHERE behave like an INNER JOIN?
4. Here is an EXPLAIN ANALYZE: why is it slow and what index would you add?
5. Which column order for an index on (org\_id, status, created\_at), and why?
6. Explain a lost update and three ways to prevent it
7. What causes a deadlock and how do you avoid one?
8. What does ON DELETE SET NULL on tasks.assignee\_id protect against?
9. When would you drop from Prisma to raw SQL?
10. Embed or reference: comments on a task, and an activity log?
11. Why is TeamForge split across Postgres and Mongo, and what does the split cost?
12. A read replica returns stale data right after a write: why, and what are your options?

**Exit criteria (all true = Module 2 done)**

- ☐ You answered the 12 questions without notes
- ☐ schema.sql, seed.sql, indexes, three transactions and schema.prisma all exist in TeamForge and run cleanly
- ☐ ActivityLog and Notification run on Mongo with a compound index, aggregation feed and TTL
- ☐ The performance notes and the decision record are written
- ☐ You backed up and restored both databases

## Deferred (consciously skipped: don't chase)

PL/pgSQL, stored procedures and triggers; full-text search; setting up Postgres partitioning or replication; PgBouncer configuration; row-level security (revisit in Module 3 security, it matters for multi-tenancy); extensions such as PostGIS and pgvector; Mongo sharding setup and Atlas Search; graph and columnar databases; vacuum tuning and planner internals.

## Resources (only these)

PostgreSQL official tutorial and SQL reference · Use The Index, Luke (indexing) · Prisma docs (schema, migrate, transactions, raw queries) · MongoDB docs: data modelling, indexes, aggregation · Optional: Designing Data-Intensive Applications, chapters 2, 3 and 7, skimmed
