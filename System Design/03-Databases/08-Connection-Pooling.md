# Connection Pooling

## 1. Simple Explanation

Opening a DB connection is expensive. A **connection pool** keeps a set of reusable connections so NestJS requests borrow/return instead of connect/disconnect each time.

## 2. Practical Backend Example

1000 concurrent HTTP requests to Nest with `pg` default without a pool can open too many Postgres connections and crash DB (`too many connections`).
Pool size 20–50 per instance (tuned) protects Postgres.

## 3. How It Works (step by step)

1. App starts pool with min/max connections.
2. Request borrows connection.
3. Runs query.
4. Returns connection to pool.
5. If pool exhausted, request waits/fails fast.

## 5. Why Do We Need It?

Postgres has a hard max_connections. App servers horizontally scale faster than DB can accept raw connections.

## 6. Alternatives & Trade-offs

| Approach | Notes |
|----------|-------|
| App pool (pg/TypeORM/Prisma) | First line of defense |
| RDS Proxy / PgBouncer | Pooling for many app tasks |
| Unlimited connections | Outage waiting to happen |

## 7. Interview Revision Notes

- Pool per process × tasks must fit DB max.
- Formula intuition: `instances * poolSize < max_connections` (leave headroom).
- Use RDS Proxy when huge task counts.

## 8. Common Interview Q&A

**Q1. Why pool connections?**
Reuse expensive connections; protect DB from connection storms.

**Q2. What happens if pool too large?**
DB overload; memory pressure; failures under traffic.

## 9. Practical Production Scenario

ECS scales to 40 tasks × pool 50 = 2000 connections > Postgres max 400 → outage. Fix: lower pool + RDS Proxy.

## 10. Self-Check Questions

1. Why is connect-per-request bad?
2. How does horizontal scale interact with pools?
3. Role of PgBouncer/RDS Proxy?
