# SQL Interview Prep — ICICI Node.js Backend (First Technical Round)

Practical SQL for banking-style schemas. Prefer clear, correct answers with a short query example.

---

### Q1. What is a Primary Key (PK)? `[Critical]`

**Answer:** A primary key uniquely identifies each row in a table. It must be UNIQUE and NOT NULL. A table has one primary key, which can be a single column or a composite of columns. Databases use the PK for lookups and as the target of foreign keys.

**Example:**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) NOT NULL UNIQUE
);
```

**Follow-up:** Surrogate keys (UUID/serial) vs natural keys (email) — banks often use surrogate PKs and unique constraints on business identifiers.

---

### Q2. What is a Foreign Key (FK)? `[Critical]`

**Answer:** A foreign key is a column (or set of columns) that references a primary/unique key in another table. It enforces referential integrity so you cannot point to a missing parent row. Actions like `ON DELETE RESTRICT/CASCADE/SET NULL` define what happens when the parent is deleted.

**Example:**
```sql
CREATE TABLE accounts (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  CONSTRAINT accounts_user_id_fkey
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE RESTRICT
);
```

**Follow-up:** For money tables, prefer `RESTRICT`/`NO ACTION` over `CASCADE` so deletes cannot silently wipe balances.

---

### Q3. What is an index, and why do indexes matter? `[Critical]`

**Answer:** An index is a data structure (commonly B-tree) that speeds up lookups, joins, sorting, and some filters by avoiding full table scans. Indexes speed reads but add cost to writes and use disk. Index columns used in `WHERE`, `JOIN`, and often `ORDER BY`.

**Example:**
```sql
CREATE INDEX idx_accounts_user_id ON accounts(user_id);
CREATE UNIQUE INDEX idx_users_email ON users(email);
```

**Follow-up:** Unused indexes waste write throughput — review with `pg_stat_user_indexes` / slow query logs.

---

### Q4. What is a composite index? How does column order matter? `[High]`

**Answer:** A composite index covers multiple columns in a defined left-to-right order. A query can use the leading columns efficiently; skipping the first column usually prevents a full use of that index (depending on engine and index-only/skip-scan support). Put equality columns first, then range columns, matching your hottest queries.

**Example:**
```sql
-- Good for: WHERE user_id = ? AND created_at >= ?
CREATE INDEX idx_txn_user_created ON transactions(user_id, created_at);

-- Uses index well:
SELECT * FROM transactions
WHERE user_id = '...' AND created_at >= '2024-01-01';

-- May not use the composite well if only filtering created_at
SELECT * FROM transactions WHERE created_at >= '2024-01-01';
```

**Follow-up:** Covering indexes (include all selected columns) can avoid heap lookups — useful for hot read paths.

---

### Q5. Clustered vs non-clustered indexes? `[High]`

**Answer:** A **clustered** index defines the physical order of table rows (one per table; in SQL Server the PK is often clustered; in InnoDB the PK is the clustered index). A **non-clustered** index is a separate structure pointing to rows. Postgres does not use “clustered index” the same way — its heap is unordered by default; `CLUSTER` rewrites once, and indexes are separate B-trees. InnoDB: secondary indexes store PK values as the row pointer.

**Example:**
```sql
-- MySQL InnoDB: PRIMARY KEY is clustered
CREATE TABLE ledger (
  id BIGINT PRIMARY KEY,          -- clustered
  account_id UUID NOT NULL,
  amount_cents BIGINT NOT NULL
);
CREATE INDEX idx_ledger_account ON ledger(account_id); -- secondary / non-clustered
```

**Follow-up:** Choose a PK that grows (time-ordered/serial) carefully vs random UUIDs — random PKs can cause page splits/fragmentation in clustered engines.

---

### Q6. Explain normalization: 1NF, 2NF, 3NF. `[Critical]`

**Answer:** **1NF:** atomic values, no repeating groups — one value per column/row intersection. **2NF:** 1NF + every non-key attribute depends on the *whole* composite key (no partial dependency). **3NF:** 2NF + no transitive dependency (non-key attributes depend only on the key, not on other non-keys). Normalization reduces redundancy and update anomalies.

**Example:**
```text
Bad (not 1NF): users.phones = "999,888"
Bad (not 2NF): order_items(order_id, product_id, product_name, qty)
               product_name depends only on product_id
Bad (not 3NF): employees(id, dept_id, dept_name)
               dept_name depends on dept_id

Better: separate products, departments tables; store only FKs.
```

**Follow-up:** Interviewers want the dependency language, not just “split tables.”

---

### Q7. What is denormalization, and when is it OK? `[High]`

**Answer:** Denormalization intentionally stores redundant data (e.g. cached totals, duplicated names) to speed reads or simplify queries. Use it when measured read cost dominates and you can keep copies consistent (triggers, app writes, batch jobs). In banking, prefer a ledger of immutable events plus derived balances maintained carefully — not ad-hoc duplicated money fields without rules.

**Example:**
```sql
-- Derived balance table updated in the same transaction as ledger insert
UPDATE accounts SET balance_cents = balance_cents - 500 WHERE id = $1;
INSERT INTO ledger(account_id, amount_cents, ...) VALUES ($1, -500, ...);
```

**Follow-up:** Always name the consistency strategy — same transaction, async rebuild, or CQRS read model.

---

### Q8. Explain JOIN types (INNER, LEFT, RIGHT, FULL, CROSS). `[Critical]`

**Answer:** **INNER** returns matching rows only. **LEFT** returns all left rows plus matches (NULLs if no match). **RIGHT** is the mirror of LEFT. **FULL OUTER** returns rows from both sides with NULLs where unmatched. **CROSS** is a cartesian product. Prefer INNER/LEFT in app SQL; be explicit about predicates in the `ON` clause.

**Example:**
```sql
-- Users with their accounts (users without accounts still appear)
SELECT u.id, u.email, a.id AS account_id
FROM users u
LEFT JOIN accounts a ON a.user_id = u.id;

-- Only users who have at least one open account
SELECT DISTINCT u.*
FROM users u
INNER JOIN accounts a ON a.user_id = u.id AND a.status = 'OPEN';
```

**Follow-up:** Filtering a LEFT JOINed table in `WHERE` can effectively turn it into an INNER JOIN — put those filters in `ON` if you need to keep non-matches.

---

### Q9. GROUP BY and HAVING — what’s the difference? `[Critical]`

**Answer:** `GROUP BY` aggregates rows into groups. `WHERE` filters rows *before* aggregation. `HAVING` filters groups *after* aggregation. Use `HAVING` for conditions on `COUNT`, `SUM`, etc.

**Example:**
```sql
SELECT user_id, COUNT(*) AS txn_count, SUM(amount_cents) AS total
FROM transactions
WHERE created_at >= DATE '2024-01-01'
GROUP BY user_id
HAVING COUNT(*) >= 10
ORDER BY total DESC;
```

**Follow-up:** Every selected non-aggregated column must appear in `GROUP BY` (SQL standard / Postgres).

---

### Q10. What is a subquery? Correlated vs non-correlated? `[High]`

**Answer:** A subquery is a query nested inside another. Non-correlated runs independently and returns a value/set. Correlated references the outer row and runs once per outer row (can be slow). Often rewrite correlated subqueries as joins or window functions for performance.

**Example:**
```sql
-- Non-correlated: accounts richer than average
SELECT *
FROM accounts
WHERE balance_cents > (SELECT AVG(balance_cents) FROM accounts);

-- Correlated: users who have more than 3 accounts
SELECT u.*
FROM users u
WHERE (
  SELECT COUNT(*) FROM accounts a WHERE a.user_id = u.id
) > 3;
```

**Follow-up:** `EXISTS` often beats `IN (SELECT ...)` for large nullable sets — know both.

---

### Q11. What is a CTE (Common Table Expression)? `[High]`

**Answer:** A CTE is a named temporary result defined with `WITH ... AS (...)`. It improves readability for multi-step logic and can be recursive for hierarchies. In Postgres, CTEs can act as optimization fences in older versions — still excellent for clarity.

**Example:**
```sql
WITH daily AS (
  SELECT account_id, DATE(created_at) AS d, SUM(amount_cents) AS net
  FROM transactions
  GROUP BY account_id, DATE(created_at)
)
SELECT account_id, d, net
FROM daily
WHERE net < 0
ORDER BY d DESC;
```

**Follow-up:** Recursive CTE example — org chart or category trees with `WITH RECURSIVE`.

---

### Q12. Explain window functions. `[High]`

**Answer:** Window functions compute values across related rows *without collapsing* them like `GROUP BY`. Use `OVER (PARTITION BY ... ORDER BY ...)` for ranking, running totals, and previous/next row access. Critical for statements, leaderboards, and deduplication.

**Example:**
```sql
SELECT
  id,
  account_id,
  amount_cents,
  SUM(amount_cents) OVER (
    PARTITION BY account_id
    ORDER BY created_at, id
  ) AS running_balance,
  ROW_NUMBER() OVER (
    PARTITION BY account_id
    ORDER BY created_at DESC
  ) AS rn
FROM transactions;

-- Latest txn per account
SELECT * FROM (
  SELECT t.*,
         ROW_NUMBER() OVER (PARTITION BY account_id ORDER BY created_at DESC) AS rn
  FROM transactions t
) x
WHERE rn = 1;
```

**Follow-up:** Know `ROW_NUMBER`, `RANK`, `DENSE_RANK`, `LAG`/`LEAD`, `SUM/AVG` as windows.

---

### Q13. What is a database transaction? `[Critical]`

**Answer:** A transaction groups SQL statements into one atomic unit of work — all commit or all roll back. Use transactions whenever multiple writes must succeed together (debit + credit + ledger insert). Keep transactions short to reduce lock contention.

**Example:**
```sql
BEGIN;
UPDATE accounts SET balance_cents = balance_cents - 1000 WHERE id = $1 AND balance_cents >= 1000;
UPDATE accounts SET balance_cents = balance_cents + 1000 WHERE id = $2;
INSERT INTO ledger(from_id, to_id, amount_cents) VALUES ($1, $2, 1000);
COMMIT;
-- On failure: ROLLBACK;
```

**Follow-up:** Check `UPDATE ... WHERE balance >= amount` rowcount — don’t trust unchecked balances under concurrency.

---

### Q14. Explain ACID. `[Critical]`

**Answer:** **Atomicity** — all-or-nothing. **Consistency** — constraints/invariants hold after commit. **Isolation** — concurrent transactions don’t see each other’s incomplete work beyond the isolation level. **Durability** — committed data survives crashes (WAL/fsync). Banking systems lean hard on A, C, and D; isolation is tuned for throughput vs correctness.

**Example:**
```text
Transfer: debit + credit must be Atomic.
CHECK/FK keep Consistency.
Isolation stops dirty reads of half-finished transfers.
Durability means a committed payment is not lost after restart.
```

**Follow-up:** “Consistency” here is DB constraints/state rules — not CAP theorem consistency.

---

### Q15. What are isolation levels? `[Critical]`

**Answer:** Isolation levels control what anomalies concurrent transactions can see. From weaker to stronger (typical SQL standard): **Read Uncommitted** → **Read Committed** → **Repeatable Read** → **Serializable**. Stronger levels reduce anomalies but increase locking/abort rates. Postgres default is Read Committed; MySQL InnoDB default is Repeatable Read.

**Example:**
```sql
-- Postgres
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- critical balance logic
COMMIT;

SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

**Follow-up:** Pick the weakest level that still preserves your invariant — often Read Committed + row locks (`FOR UPDATE`) for account balances.

---

### Q16. Dirty read, non-repeatable read, phantom read? `[Critical]`

**Answer:** **Dirty read:** see uncommitted data from another transaction (can vanish on rollback). **Non-repeatable read:** re-read the same row and see a committed change by another txn. **Phantom read:** re-run a range query and see new/missing rows from another txn’s inserts/deletes. Serializable aims to prevent all three; weaker levels allow some.

**Example:**
```text
Dirty:     T1 updates balance, T2 reads it, T1 rolls back → T2 saw fiction
Non-rep:   T1 reads balance 100; T2 commits debit; T1 reads 50
Phantom:   T1 counts OPEN accounts = 3; T2 inserts new OPEN; T1 counts 4
```

**Follow-up:** Map which isolation levels allow which anomaly — classic interview table.

| Anomaly            | Read Uncommitted | Read Committed | Repeatable Read | Serializable |
|--------------------|------------------|----------------|-----------------|--------------|
| Dirty read         | Possible         | No             | No              | No           |
| Non-repeatable     | Possible         | Possible       | No*             | No           |
| Phantom            | Possible         | Possible       | Possible*       | No           |

\*Engine-specific nuances (InnoDB RR prevents many phantoms via next-key locks; Postgres RR is snapshot-based).

---

### Q17. What are deadlocks? How do you handle them? `[High]`

**Answer:** A deadlock happens when transactions wait on each other’s locks in a cycle. The DB detects it and aborts one transaction. Prevent by locking resources in a consistent order (e.g. always lock the lower account id first), keeping transactions short, and retrying the aborted transaction safely/idempotently.

**Example:**
```sql
-- Always lock accounts in sorted id order
SELECT * FROM accounts
WHERE id IN ($1, $2)
ORDER BY id
FOR UPDATE;
```

**Follow-up:** App layer should catch deadlock errors and retry with backoff for idempotent operations.

---

### Q18. Optimistic vs pessimistic locking? `[High]`

**Answer:** **Pessimistic** locks the row while you work (`SELECT FOR UPDATE`) so others wait — good for high-contention balances. **Optimistic** stores a version/timestamp and updates only if the version matches; on conflict, retry — good for low contention. Banking transfers often use pessimistic locks inside a transaction; profile updates may use optimistic versions.

**Example:**
```sql
-- Pessimistic
BEGIN;
SELECT balance_cents FROM accounts WHERE id = $1 FOR UPDATE;
UPDATE accounts SET balance_cents = balance_cents - $2 WHERE id = $1;
COMMIT;

-- Optimistic
UPDATE accounts
SET balance_cents = balance_cents - $2,
    version = version + 1
WHERE id = $1 AND version = $3;
-- if rowcount = 0 → someone else won; reload and retry
```

**Follow-up:** Sequelize/TypeORM both support version columns for optimistic locking and `LOCK.UPDATE` / pessimistic lock modes.

---

### Q19. How do you optimize a slow SQL query? `[Critical]`

**Answer:** Measure first (`EXPLAIN`/`EXPLAIN ANALYZE`), ensure selective indexes match filters/joins, avoid `SELECT *`, reduce scanned rows early, rewrite bad correlated subqueries, fix accidental cartesian joins, and paginate with keyset instead of huge `OFFSET`. Check statistics are fresh (`ANALYZE`). Don’t add random indexes without evidence.

**Example:**
```sql
EXPLAIN ANALYZE
SELECT * FROM transactions
WHERE account_id = $1
ORDER BY created_at DESC
LIMIT 50;

-- Prefer keyset pagination
SELECT * FROM transactions
WHERE account_id = $1
  AND (created_at, id) < ($2, $3)
ORDER BY created_at DESC, id DESC
LIMIT 50;
```

**Follow-up:** Mention covering indexes, partial indexes (`WHERE status = 'OPEN'`), and avoiding functions on indexed columns in `WHERE` when possible.

---

### Q20. How do you read EXPLAIN output? `[High]`

**Answer:** `EXPLAIN` shows the planner’s chosen plan; `EXPLAIN ANALYZE` runs it and shows actual timings/rows. Look for sequential scans on large tables when you expected index use, nested loop joins with huge row estimates, and big mismatches between estimated vs actual rows (stale stats). Cost numbers are planner units; actual time in ANALYZE is what users feel.

**Example:**
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT u.email, COUNT(a.id)
FROM users u
LEFT JOIN accounts a ON a.user_id = u.id
WHERE u.status = 'ACTIVE'
GROUP BY u.email;
```

**Follow-up:** “Seq Scan isn’t always bad” — on tiny tables it can beat an index. Bad when millions of rows are scanned repeatedly.

---

### Q21. UNION vs UNION ALL? `[High]`

**Answer:** `UNION` concatenates result sets and removes duplicates (extra sort/hash). `UNION ALL` keeps duplicates and is cheaper. Use `UNION ALL` unless you truly need distinct rows.

**Example:**
```sql
SELECT email FROM customers
UNION ALL
SELECT email FROM prospects;
```

**Follow-up:** Column counts/types must match across branches.

---

### Q22. DELETE vs TRUNCATE vs DROP? `[Critical]`

**Answer:** **DELETE** removes rows (optional `WHERE`); can be transactional/logged row-by-row; fires triggers; FK-checked. **TRUNCATE** empties a table quickly (deallocates pages); usually cannot selectively filter; in many engines minimal logging; may reset identity; often blocked by FKs. **DROP** removes the table (or DB object) definition and data entirely.

**Example:**
```sql
DELETE FROM sessions WHERE expires_at < NOW();  -- selective
TRUNCATE TABLE staging_imports;                 -- wipe all rows fast
DROP TABLE staging_imports;                     -- remove table
```

**Follow-up:** Production caution — `TRUNCATE`/`DROP` are destructive; prefer migrations and backups. `DELETE` without `WHERE` still leaves the table structure.

---

### Q23. What are unique constraints vs unique indexes? `[Medium]`

**Answer:** Both enforce uniqueness. A unique constraint is an integrity rule; engines typically implement it with a unique index. You can also create a unique index directly. Partial unique indexes (Postgres) enforce uniqueness only for a subset of rows — useful with soft deletes.

**Example:**
```sql
ALTER TABLE users ADD CONSTRAINT users_email_key UNIQUE (email);

-- Soft-delete friendly (Postgres)
CREATE UNIQUE INDEX users_email_active_uq
ON users(email)
WHERE deleted_at IS NULL;
```

**Follow-up:** Composite unique constraints model business rules like “one salary account type per user.”

---

### Q24. NULL handling in SQL — common pitfalls? `[High]`

**Answer:** `NULL` means unknown. `= NULL` is always unknown — use `IS NULL` / `IS NOT NULL`. Aggregates ignore NULLs (`SUM` of empty set is NULL, `COUNT(col)` skips NULLs, `COUNT(*)` counts rows). In unique constraints, multiple NULLs are often allowed (engine-specific).

**Example:**
```sql
SELECT * FROM users WHERE middle_name IS NULL;
SELECT COUNT(*) AS rows, COUNT(phone) AS with_phone FROM users;
COALESCE(phone, 'N/A');
```

**Follow-up:** `NOT IN (subquery with NULL)` can return empty unexpectedly — prefer `NOT EXISTS`.

---

### Q25. Indexes on low-cardinality columns — useful or not? `[Medium]`

**Answer:** A standalone index on a boolean/status with 2–3 values is often unused because a large fraction of the table matches. Prefer composite indexes that combine status with a selective column, or partial indexes for the rare value (`WHERE status = 'LOCKED'`).

**Example:**
```sql
CREATE INDEX idx_accounts_locked ON accounts(user_id)
WHERE status = 'LOCKED';
```

**Follow-up:** Cardinality + selectivity beat “index every column” cargo cult.

---

### Q26. What is a covering index / index-only scan? `[Medium]`

**Answer:** If an index contains all columns needed by a query, the engine may answer from the index without touching the table heap (index-only scan in Postgres, subject to visibility map). This cuts I/O for hot read queries.

**Example:**
```sql
-- Query only needs user_id + created_at + amount
CREATE INDEX idx_txn_cover
ON transactions(account_id, created_at DESC, amount_cents);

SELECT created_at, amount_cents
FROM transactions
WHERE account_id = $1
ORDER BY created_at DESC
LIMIT 20;
```

**Follow-up:** Don’t make every index covering — write amplification grows.

---

### Q27. How do transactions interact with DDL? `[Medium]`

**Answer:** In Postgres, most DDL is transactional (can run inside a transaction and roll back). In MySQL, DDL often causes implicit commits. Know your dialect before wrapping migrations with app transactions. Long DDL locks can block production traffic — use online/schema-change strategies for large tables.

**Example:**
```sql
-- Postgres
BEGIN;
ALTER TABLE users ADD COLUMN status VARCHAR(20) DEFAULT 'ACTIVE';
UPDATE users SET status = 'ACTIVE' WHERE status IS NULL;
COMMIT;
```

**Follow-up:** Expanding locks vs rewriting tables — mention `pt-online-schema-change` / `gh-ost` / Postgres `CREATE INDEX CONCURRENTLY` as awareness.

---

### Q28. Practical: find duplicate emails. `[High]`

**Answer:** Group by the column and keep groups with `HAVING COUNT(*) > 1`. Use a window function when you need every duplicate row, not just the key.

**Example:**
```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

SELECT *
FROM (
  SELECT u.*,
         COUNT(*) OVER (PARTITION BY email) AS cnt
  FROM users u
) t
WHERE cnt > 1;
```

**Follow-up:** Fix with a unique constraint after cleaning data.

---

### Q29. Practical: second-highest salary / Nth value. `[High]`

**Answer:** Prefer `DENSE_RANK` or `ROW_NUMBER` over brittle `MAX`/`LIMIT` tricks when ties matter. State how ties should behave.

**Example:**
```sql
SELECT salary
FROM (
  SELECT salary,
         DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
  FROM employees
) t
WHERE rnk = 2;
```

**Follow-up:** `DISTINCT salary ORDER BY salary DESC OFFSET 1 LIMIT 1` works for simple cases but is weaker on ties/portability.

---

### Q30. Practical: users with no accounts (anti-join). `[High]`

**Answer:** Use `NOT EXISTS`, `LEFT JOIN ... WHERE right.id IS NULL`, or `EXCEPT`. `NOT EXISTS` is usually clear and NULL-safe.

**Example:**
```sql
SELECT u.*
FROM users u
WHERE NOT EXISTS (
  SELECT 1 FROM accounts a WHERE a.user_id = u.id
);

-- equivalent pattern
SELECT u.*
FROM users u
LEFT JOIN accounts a ON a.user_id = u.id
WHERE a.id IS NULL;
```

**Follow-up:** Prefer `NOT EXISTS` over `NOT IN` when the subquery can return NULL.

---

### Q31. Practical: running daily totals and statement query. `[Medium]`

**Answer:** Combine date truncation with window `SUM` for statement-style running balances per account.

**Example:**
```sql
SELECT
  account_id,
  created_at,
  amount_cents,
  SUM(amount_cents) OVER (
    PARTITION BY account_id
    ORDER BY created_at, id
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS balance_after
FROM transactions
WHERE account_id = $1
  AND created_at >= $2
  AND created_at < $3
ORDER BY created_at, id;
```

**Follow-up:** Immutable append-only ledger + computed balance is an interview-friendly banking design answer.

---

### Q32. Practical: pagination that doesn’t get slower. `[High]`

**Answer:** Deep `OFFSET` forces the DB to scan/skip many rows. Keyset (seek) pagination uses a cursor on the sorted key (`WHERE (created_at, id) < (?, ?)`). Stable ordering needs a unique tie-breaker (`id`).

**Example:**
```sql
-- Avoid for page 10,000
SELECT * FROM transactions ORDER BY id DESC OFFSET 200000 LIMIT 50;

-- Prefer
SELECT * FROM transactions
WHERE id < $1
ORDER BY id DESC
LIMIT 50;
```

**Follow-up:** Cursor APIs map cleanly to keyset pagination in Node services.

---

### Q33. What happens if you forget a JOIN condition? `[Medium]`

**Answer:** You get a cartesian product — every left row paired with every right row — which explodes row counts and can take down a service. Always verify join predicates and expected cardinality in code review and EXPLAIN.

**Example:**
```sql
-- Dangerous
SELECT * FROM users, accounts; -- CROSS JOIN

-- Intended
SELECT * FROM users u
JOIN accounts a ON a.user_id = u.id;
```

**Follow-up:** Sudden huge temp files / memory in EXPLAIN ANALYZE often points here.

---

### Q34. Stored procedures vs app-side SQL — what do banks sometimes prefer? `[Medium]`

**Answer:** Procedures keep logic close to data and can reduce round-trips; app-side SQL (via Sequelize/TypeORM) keeps business logic in versioned Node code and is easier for most Node teams to test. Many modern Node backends keep orchestration in services and use SQL/ORM for persistence; critical batch jobs may still use SQL functions. Be ready to say you can do both and choose based on ops standards.

**Example:**
```sql
-- Simple SQL function sketch (Postgres)
CREATE OR REPLACE FUNCTION transfer(from_id UUID, to_id UUID, amt BIGINT)
RETURNS void AS $$
BEGIN
  UPDATE accounts SET balance_cents = balance_cents - amt WHERE id = from_id;
  UPDATE accounts SET balance_cents = balance_cents + amt WHERE id = to_id;
END;
$$ LANGUAGE plpgsql;
```

**Follow-up:** If the JD is Node + Sequelize, lead with transactions in Node services unless they ask for PL/pgSQL.

---

### Q35. Quick checklist before you say “query is fine.” `[Critical]`

**Answer:** Confirm correctness first (joins, NULLs, duplicates), then concurrency (transaction + locking), then performance (`EXPLAIN ANALYZE`, indexes, limits). For financial updates: row lock or version check, constraint validation, and an immutable audit/ledger row.

**Example:**
```text
1) Right rows?  2) Safe under concurrency?  3) Indexed access path?
4) Bounded result set?  5) Idempotent retry if deadlock/serialization failure?
```

**Follow-up:** This checklist shows senior judgment even if syntax is simple.

---

## Cheat sheet

```sql
-- Keys & indexes
PRIMARY KEY | FOREIGN KEY | UNIQUE | CREATE INDEX | CREATE UNIQUE INDEX

-- Joins
INNER JOIN | LEFT JOIN | CROSS JOIN

-- Filter / aggregate
WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT

-- Set ops
UNION ALL  -- prefer unless distinct needed
UNION      -- distinct

-- Destructive
DELETE FROM t WHERE ...;  -- rows
TRUNCATE t;               -- all rows, keep table
DROP TABLE t;             -- remove table

-- Concurrency
BEGIN; ... COMMIT; | ROLLBACK;
SELECT ... FOR UPDATE;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED | REPEATABLE READ | SERIALIZABLE;

-- Plan
EXPLAIN ANALYZE SELECT ...;
```

**Common interview traps:** `WHERE` vs `HAVING`, `UNION` vs `UNION ALL`, `DELETE` vs `TRUNCATE` vs `DROP`, NULLs with `NOT IN`, OFFSET pagination, and claiming Serializable is “always required” without discussing throughput.
