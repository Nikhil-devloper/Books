# CAP Theorem

## 1. Simple Explanation

In a distributed system (data on multiple nodes), when a **network partition** happens, you cannot have both perfect **Consistency** and perfect **Availability** at the same time. You choose.

- **C (Consistency):** every read sees the latest successful write.
- **A (Availability):** every request gets a non-error response (may be stale).
- **P (Partition tolerance):** system continues despite network splits.

On real networks, **P is required**. So practical choice is **CP vs AP** during partition.

## 2. Practical Backend Example

Bank account balance service across two AZs:
- CP mindset: if replica cannot confirm, fail the read/write rather than show wrong balance.
Social like-count:
- AP mindset: keep serving; counts may be slightly wrong temporarily.

## 3. How It Works (step by step)

1. Data replicated across nodes.
2. Network between nodes breaks (partition).
3. If you allow writes on both sides → inconsistency.
4. If you block writes/reads until quorum → lose some availability.
5. After partition heals, resolve conflicts / catch up replicas.

## 5. Why Do We Need It?

Helps you explain product trade-offs: money vs likes need different defaults.

## 6. Alternatives & Trade-offs

| Choice | Behavior during partition | Examples |
|--------|---------------------------|----------|
| CP | Reject some requests to keep correctness | Strong SQL quorum systems, many financial paths |
| AP | Serve possibly stale data | DynamoDB eventual mode, caches, social counters |
| CA | Only realistic on single-node / no partition | Single Postgres primary (no distributed P) |

## 7. Interview Revision Notes

- CAP is about partition scenarios in distributed data systems.
- Pick CP for critical correctness; AP for availability-first features.
- Don't say "we want CAP all three" — that signals misunderstanding.

## 8. Common Interview Q&A

**Q1. State CAP theorem simply.**
During a network partition, choose consistency or availability.

**Q2. Is CAP about everyday normal operation?**
It's about trade-offs when partitions happen; systems still tune latency/consistency in normal mode (see PACELC).

## 9. Practical Production Scenario

Users in two regions update the same shopping cart. CP approach may block; AP may need merge rules (last-write-wins or field-level merge).

## 10. Self-Check Questions

1. What do C, A, P mean?
2. Why is P assumed in cloud systems?
3. CP or AP for payment capture? For view-count?
