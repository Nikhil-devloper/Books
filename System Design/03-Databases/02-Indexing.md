# Database Indexing

## 1. Simple Explanation

An **index** is an extra data structure that helps the database find rows quickly without scanning the whole table — like a book index.

## 2. Practical Backend Example

`SELECT * FROM orders WHERE user_id = 42 ORDER BY created_at DESC` on 50M rows.
Without index: full table scan (seconds).
With index on `(user_id, created_at)`: milliseconds.

## 3. How It Works (step by step)

1. DB stores index keys pointing to rows (often B-tree).
2. Query planner uses index if selective enough.
3. Writes must update indexes too → more write cost.
4. Too many indexes slow inserts/updates.
5. Use `EXPLAIN ANALYZE` to verify.

## 5. Why Do We Need It?

Latency and throughput of reads depend on indexes matching real queries.

## 6. Alternatives & Trade-offs

| Index type | Use |
|------------|-----|
| B-tree | Equality/range (default) |
| Hash | Equality (limited) |
| GIN | JSONB/arrays/full text (Postgres) |
| Composite | Multi-column filters/sorts |

## 7. Interview Revision Notes

- Index for query patterns, not every column.
- Leftmost prefix rule for composite indexes.
- Write amplification vs read speed trade-off.

## 8. Common Interview Q&A

**Q1. How do indexes speed reads?**
Allow seeking to matching rows instead of scanning all.

**Q2. Downside?**
Extra storage + slower writes + planner complexity.

## 9. Practical Production Scenario

p99 spike on search-by-email. `EXPLAIN` shows seq scan. Add unique index on `lower(email)`; p99 drops from 800ms to 15ms.

## 10. Self-Check Questions

1. What is a full table scan?
2. Why not index every column?
3. Design a composite index for user_id + created_at DESC.
