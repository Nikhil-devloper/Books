# SQL vs NoSQL

## 1. Simple Explanation

**SQL (relational):** tables, rows, schemas, powerful queries/joins, strong transactions (e.g. PostgreSQL).
**NoSQL:** varied models (document, key-value, wide-column). Often flexible schema + easy horizontal scale (e.g. DynamoDB).

## 2. Practical Backend Example

Orders with payments/line items needing transactions → PostgreSQL.
User session store / ultra-high scale key lookup → DynamoDB or Redis.
Product catalog with varied attributes → could be either; pick based on query patterns.

## 3. How It Works (step by step)

1. List access patterns (reads/writes/queries).
2. Need multi-row ACID? lean SQL.
3. Need massive scale with known key lookups? lean DynamoDB-style.
4. Model data to queries (especially NoSQL).
5. Revisit indexes and sharding needs.

## 5. Why Do We Need It?

Wrong DB choice creates pain: joins hell in NoSQL, or write-scaling pain in a single SQL primary.

## 6. Alternatives & Trade-offs

| | PostgreSQL | DynamoDB |
|--|------------|----------|
| Schema | Fixed-ish | Flexible items |
| Transactions | Excellent | Limited scope |
| Scale writes | Vertical then shard | Partitions by design |
| Queries | Rich SQL | Key/condition + GSIs |

## 7. Interview Revision Notes

- Choose based on access patterns + consistency needs.
- SQL default for complex relationships.
- DynamoDB shines with known partition keys and scale.

## 8. Common Interview Q&A

**Q1. When pick Postgres over DynamoDB?**
Complex joins, ad-hoc queries, strong multi-row transactions.

**Q2. When DynamoDB?**
High scale key-value/document access patterns known up front.

## 9. Practical Production Scenario

Analytics team wants arbitrary queries on DynamoDB orders. Painful. Stream to warehouse/SQL for analytics; keep Dynamo for OLTP path.

## 10. Self-Check Questions

1. One reason for SQL.
2. One reason for NoSQL.
3. What must you know before DynamoDB data modeling?
