[Postgres for Backend Engineers](https://faqsnfacts.netlify.app/postgresql-for-backend-engineers/)

Every backend app revolves around one fundamental entity: **'Data'**

> The Core emphasis of PostgreSQL in treating data and managing it is uncompromising data integrity, reliability, and extensibility anchored in a robust relational framework

> Data must be consistent and durable

"*Working with postgres or any other DB as backend engineer, the main goal is to not only manage data and operate it, but to make the operations\queries and DB architecture optimised, efficient, and reduce costs and latency, knowing how to scale the architecture efficiently is a cherry on top*"

#### Strengths of Postgres:

- Strong traditional relational DBMS with modern features associated with NoSQL systems
- Data integrity, consistency, reliability, durability in a robust manner
- Strong relationships among different datasets/tables, consistent and durable data
- Robust and Powerful internal architecture with emphasis upon efficient optimization
- Efficient queries and indexing
- ACID properties and Transactional data operations
- Native JSON and JSONB support
- Strong backend support using ORMs

#### Tradeoffs:

- Only scales vertically \(Adding physical hardware components: RAM, CPU\)
- Less flexible than NoSQL DB's, NoSQL can scale horizontally \(sharding\)
- Strict internal architecture can add latency overhead
- Purpose-specific databases like Redis for caching exist where postgres cannot be a practical good choice

#### Relational Databases Core:

- Syntax
- Constraints
- Schema Design
- Tabular Architecture
- Normalization & Denormalization

## What matters in learning and working with PostgreSQL?
---

- ⭐ Bare minimums \(50%\)
    - Syntax
    - datatypes
    - constraints
    - schema design
    - writing queries manually and naturally, being able to reason and understand logic
    - understanding hierarchy and anatomy of postgres \(CLI, GUI, under the hood\)
    - users and roles

- Knowing the internal architecture \(40%\)
    - under the hood execution steps/process: `EXPLAIN` and `EXPLAIN ANALYZE`
    - MVCC
    - WAL
    - Autovacuum
    - ⭐ How postgres actually manages data in physical memory \(Memory = RAM, Storage = Disk\)
    - Shared memory - shared_buffers
    - ⭐ process management \(MySQL: one single process with lightweight threads. PostgreSQL: multiple independent OS processes\), process-per-connection via TCP
    - ⭐ connections: types\(localhost, socket\) - pooling\(PgBouncer\) / connecting to a backend \(ORM\)

- Tooling \(10%\)
    - pgadmin, psql etc

- Monitoring and Observability \(After completing previous concepts and finally working with backend apps\)
    - there are chances you'll be missing out on some stuff, observability allows you to detect how things work and what can be done, possibly exposing you to something new and important
    - Database backups
    - Critical cases


### Your 7-day calendar (90–120 min/day)

| Day | Topic | Details | Deliverable |
| :--- | :--- | :--- | :--- |
| **Day 01** | Environment + relational foundations | Setup, PostgreSQL hierarchy, `psql`, data types, constraints and your first schema. | Build TeamForge's initial database schema. |
| **Day 02** | SQL fundamentals | `SELECT`, `WHERE`, filtering, ordering, `NULL`, expressions, `CASE` and `INSERT` basics. | Write simple queries without looking up syntax. |
| **Day 03** | Relational querying | `JOIN`s, aggregations, `GROUP BY`, `HAVING`, subqueries, `EXISTS` and CTEs. | Solve real multi-table data problems. |
| **Day 04** | Database design + transactions | Normalization, relationship design, `UPDATE`, `DELETE`, `RETURNING`, transactions and indexes. | Make your schema reliable and modify data safely. |
| **Day 05** | PostgreSQL architecture + connections | Client-server model, processes, sockets, TCP, authentication, connections and pooling. | Trace a database connection from Node.js to PostgreSQL. |
| **Day 06** | Memory + PostgreSQL internals | RAM, disk pages, shared buffers, MVCC, WAL, autovacuum and query execution plans. | Understand how data is stored, modified, recovered and retrieved. |
| **Day 07** | Node integration + revision | Node.js driver, connection pools, parameterized queries and a comprehensive SQL exercise. | Put the concepts together in a small working system. |