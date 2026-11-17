### Day 1
---

1. Setup postgres: psql and pgadmin
    - explore psql commands
    - understand postgres hierarchy
    - understand process mgmt, cluster, database, connections

2. Understand data modelling: must-to-do stuff before writing a single query
    - understanding database requirements: what are we trying to achieve
    - what are the entities, and their attributes
    - relationships among entities
    - ...overall DB architecture

3. Actually writing queries: STANDARD COMMANDS (;) and psql meta commands (\\l, \\dt etc.)
    - CREATE DATABASE, \c db_name
    - CREATE TABLE
    - INSERT INTO table VALUES
    - Datatypes and Foreign Keys
    - The SELECT command
    - ALTER TABLE commands: ADD COLUMN, ADD PRIMARY KEY, REFERENCES ...
    - "" vs ''
    - UPDATE table

4. Understanding types of relationships:
    - 1:1, 1:N, M:N (junction tables), Self Referencing (recursive)

5. Types of Keys:
    - Primary Key: non repeating/duplicate, not null, unique identifier (eg. userID)
    - Foreign Key: store the primary key of another table to create a link/relationship, can be null (eg. userId column in Orders Table)
    - Candidate Key: a column which proves uniqueness even if it isnt chosen as the PK. (eg. email)
    - Composite Key: a primary key made up of multiple foreign keys, which uniquely identifies a row (userID, ticketID)

6. Junction tables: for many-to-many relationships, these dont really need a separate standalone primary key column, every record uses the combination of Foreign keys as a composite key

7. Referential Integrity: security mechanism, orphaned records case: RESTRICT, CASCADE DELETE, SET NULL
    - FK: ON DELETE CASCADE

8. Normalization (1,2,3 NF): the technique focused upon ensuring data integrity, such that data and relationships stay strong, concise and robust in many conditions/cases 
    - organizing and structuring data: preventing insconsistent data and reduce redundancy
    - relationships must only be with reference of *just one key*
    - **1NF**: every column must hold a single atomic(no arrays) unique(non-repeating) value, no duplicate rows and columns
    - **2NF**: must be 1NF, only works with tables with a composite PK (multiple FKs) - junction tables. Every non ket attr must depend upon all the composite keys, not just half or few of them
    - **3NF**: must be 2NF, no non-key attr must be dependent on another non-key attr
    - **tradeoff**: too much normalization = too much JOIN operations = slows the system
    - Most transactional applications aim for Third Normal Form (3NF) to balance consistency and performance


### Day 2: DataTypes and Constraints (datatypes.md), Joins (joins.md)