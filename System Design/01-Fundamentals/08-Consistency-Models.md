# Consistency Models

## 1. Simple Explanation

**Consistency model** = rules about when a read sees a write.

Common models:
- **Strong consistency:** read always returns the latest committed write.
- **Eventual consistency:** if no new writes, replicas eventually agree; reads may be stale briefly.
- **Read-your-writes:** a user sees their own updates immediately.
- **Causal consistency:** if A caused B, everyone sees A before B.

## 2. Practical Backend Example

After updating profile name in NestJS:
- Strong: next GET from any node shows new name (e.g. read from primary).
- Eventual: read replica may show old name for hundreds of ms.
- Read-your-writes: that user sees new name (read primary or sticky), others may lag.

## 3. How It Works (step by step)

1. Write goes to primary (or leader).
2. Replicas apply changes asynchronously or synchronously.
3. Sync replication → stronger consistency, higher write latency.
4. Async replication → faster writes, possible stale reads.
5. App chooses read-after-write path for sensitive flows.

## 5. Why Do We Need It?

Wrong consistency causes double charges, duplicate orders, or confusing UI. Right consistency avoids over-engineering.

## 6. Alternatives & Trade-offs

| Model | Pros | Cons |
|-------|------|------|
| Strong | Correctness | Latency / availability cost |
| Eventual | Speed, availability | Stale reads, conflict handling |
| Read-your-writes | Good UX for actor | Extra routing logic |

## 7. Interview Revision Notes

- Match consistency to business risk.
- Payments/inventory: stronger. Feeds/counters: often eventual.
- Say "read from primary after write" as a practical pattern.

## 8. Common Interview Q&A

**Q1. Strong vs eventual?**
Strong = always latest. Eventual = temporarily stale, converges later.

**Q2. How to get read-your-writes on RDS?**
Route that user's reads to primary for a short window, or use synchronous session read.

## 9. Practical Production Scenario

Order status shows "pending" after success because UI read a lagging replica. Fix: read primary for order-by-id right after write, replicas for order history lists.

## 10. Self-Check Questions

1. Define eventual consistency.
2. When is stale data unacceptable?
3. One app-level trick for read-your-writes?
