# Distributed Locking

## 1. Simple Explanation

A **distributed lock** ensures only one worker across many machines performs a critical section at a time (e.g. Redis lock, DB advisory lock).

## 2. Practical Backend Example

Cron on 5 ECS tasks all try to generate monthly invoices. Redis lock `lock:invoices:2026-11` so only one runs.

## 3. How It Works (step by step)

1. Try to acquire lock (SET NX EX).
2. If got lock, do work.
3. Release lock (safely, with token).
4. Lock auto-expires to survive crashes.

## 5. Why Do We Need It?

Prevent duplicate side effects under multi-instance deployments.

## 6. Alternatives & Trade-offs

| Approach | Notes |
|----------|-------|
| Redis lock | Fast; careful with expiry |
| DB unique job row | Simple correctness |
| Kafka single partition key | Ordering/single consumer |

## 7. Interview Revision Notes

- Prefer idempotency over heavy locking when possible.
- Always set lock TTL.
- Avoid long lock holds.

## 8. Common Interview Q&A

**Q1. Why distributed lock?**
Coordinate mutual exclusion across nodes.

**Q2. Danger?**
Deadlocks, long TTLs, assuming lock = safety without idempotency.

## 9. Practical Production Scenario

Two nodes send the same push notification. Add lock + idempotency key.

## 10. Self-Check Questions

1. What is SET NX EX?
2. Why TTL on locks?
3. Alternative to locks for jobs?
