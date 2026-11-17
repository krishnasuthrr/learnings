### Datatypes and Constraints
---

1. Identifiers (uniquely identify rows in tables): INTEGER, BIGINT, and UUID
    1. **INTEGER(4B), BIGINT(8B)**: sequential, numeric, auto incrementing.
        - Adv: small, fast indexing
        - Disadv: predictable due to being sequential, vulnerable if exposed to apis/urls
    2. **UUID(16B0)**: eg. 550e8400-e29b-41d4-a716-446655440000 - generated cryptographically, chances of getting the same uuid is practically 0
        - Adv: complex, unpredictable, hence safe for apis and urls
        - Disadv: larger than integers, occupy more space in disk, slower R/W, indexing large number of rows becomes slow and performance heavy

2. Characters / Strings: VARCHAR, TEXT: almost no diff in perf. and space they're both same internally
    1. **TEXT**: string of any length
    2. **VARCHAR or VARCHAR(n)**: 'n' is the string lenght limit

3. Decimals (REAL, NUMERIC, DECIMAL):
    - NUMERIC and DECIMAL guarantee 100% mathematical accuracy but take up more space

4. Time
    - **TIMESTAMP**: timestamp without timezone
    - **TIMESTAMPTZ**: timestamp with timezone (industry standard)

5. JSON: NoSQL flexibility: (JSONB: industry standard, use it 99% of the time)
    - **JSON**: stores text strings as they are, preserves white spaces and duplicate keys
                slow operations due to string parsing
    - **JSONB**: postgres parses json during insertion: converts into binary, strips out whitespaces and duplicates
                fast Read operations and indexing, slow writing but that's acceptable

### Constraints
---

the rules, security guards of data, even if your app has data validations, alwyas ensure data security at DB level
applied on columns, tables

- PRIMARY KEY
- FOREIGN KEY
- NOT NULL
- UNIQUE: no two rows share the same value
- DEFAULT \<value\>
- BOOLEAN
- CHECK (expression): Boolean, must evaluate to TRUE
- foreign-key deletion behaviour: CASCADE, SET NULL, RESTRICT and NO ACTION
- DISTINCT: select/return only distinct/unique values and not repeat duplicates