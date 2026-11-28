# System Design Interview Framework (7+ YOE)

Use this sequence in every high-level design interview (~45–60 min).

## The 10-step sequence

1. **Clarify functional requirements** — what features are in scope?
2. **Clarify non-functional requirements** — latency, availability, consistency, scale.
3. **Estimate traffic and storage** — QPS, read/write ratio, data size, growth.
4. **Define APIs** — main endpoints, payloads, auth.
5. **Design the database schema** — tables/collections, keys, indexes.
6. **Draw high-level architecture** — clients, LB, services, cache, DB, queue, CDN.
7. **Explain request/data flow** — one happy path end-to-end.
8. **Identify bottlenecks** — CPU, DB, network, hot keys, fan-out.
9. **Discuss scaling strategies** — replicas, shards, cache, async, partitions.
10. **Discuss failures and trade-offs** — SPOF, retries, idempotency, consistency.

## Time boxing (60 min)

| Phase | Minutes |
|-------|---------|
| Clarify FR + NFR | 5–8 |
| Estimates | 5 |
| APIs + schema | 8–10 |
| High-level design + flow | 15 |
| Deep dive (bottlenecks/failures) | 15–20 |
| Wrap trade-offs | 3–5 |

## What seniors do differently

- Ask scope questions before drawing boxes.
- Use rough numbers (orders of magnitude are fine).
- Justify choices: "I chose X because Y; downside is Z."
- Prefer simple designs first, then scale the bottleneck.
- Call out consistency needs explicitly (payment vs like-count).
- Design for failure: retries, DLQ, timeouts, circuit breakers.

## What weak answers look like

- Dumping buzzwords: Redis, Kafka, microservices — with no why.
- Jumping to microservices for a simple CRUD app.
- Ignoring data model and indexes.
- No failure discussion.
- No capacity estimate.

## Back-of-envelope cheat sheet

- Seconds/day ≈ 86,400 ≈ **10^5**
- 1M users × 10 actions/day ≈ 10M/day ≈ **100 QPS** average
- Peak often **3–10×** average
- 1 KB × 1B rows ≈ **1 TB**
- In-memory Redis: plan for working set, not full DB

## Senior mantra

> Clarify → Estimate → APIs → Data → Architecture → Flow → Bottlenecks → Failures → Trade-offs
