### PostgreSQL hierarchy
---

Server
Cluster
Database
Schema
Tables

---

**Server**: The actual Physical/Virtual machine/container on which postgres database is hosted, more specifically and accurately, a server in postgres is the actual background process or daemon `postgres` which manages all DB operations, connections etc.
    - Postgres has a multi process architecture under the hood, with one main master father-of-all parent process - the `postmaster`
    - each process serves a specific purpose
