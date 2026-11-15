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

- Bare minimums
    - Syntax
    - datatypes
    - constraints
    - schema design
    - writing queries manually
    - understanding hierarchy and anatomy of postgres \(CLI, GUI, under the hood\)

- Knowing the internal architecture
    - under the hood execution steps/process: `EXPLAIN` and `EXPLAIN ANALYZE`
    - MVCC
    - WAL
    - Autovacuum
    - How postgres manages data actually in physical memory
    - Shared memory - shared_buffers
    - process management
    - ⭐ connections: types\(localhost, socket\) - pooling / connecting to a backend \(ORM\)

- Tooling
    - pgadmin, psql etc

