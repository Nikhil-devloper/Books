# 13 — System Design (30-min First Round Level)

Keep answers **practical and small**. Interviewer wants clear building blocks for a Node.js banking/API backend — not Kafka-everywhere distributed systems essays.

---

### Q1. How does load balancing work in front of Node.js? `[Critical]`

**Answer:**
A load balancer (NGINX, ALB, HAProxy) sits in front of multiple Node instances and spreads HTTP traffic. Common strategies: round-robin, least connections. Health checks remove unhealthy instances. For sticky sessions you need cookie affinity — better to keep the app **stateless** so any instance can serve any request. TLS often terminates at the LB.

**Follow-up:** What if one instance is slow but still “healthy”? — Add latency-aware checks / circuit at LB; fix the instance (CPU, pool, GC).

---

### Q2. Horizontal vs vertical scaling — which for Node.js? `[Critical]`

**Answer:**
**Vertical** = bigger machine (more CPU/RAM). Simple, but has a ceiling and single point of failure. **Horizontal** = more Node processes/pods behind a LB. Node is single-threaded per process, so horizontal (and multi-process on one box) is the natural fit. Prefer horizontal for availability; use vertical only for quick relief or memory-heavy workers. Stateless design + shared Redis/DB unlocks horizontal scale.

**Follow-up:** When vertical still helps? — Large in-memory workloads, fewer connection-pool multipliers, simpler ops for small traffic.

---

### Q3. How would you use Redis for caching in a Node API? `[Critical]`

**Answer:**
Cache **hot, read-heavy, slightly stale-OK** data: reference data, user profile snippets, config, frequently hit GET responses. Pattern: read Redis first → on miss hit DB → set key with TTL. Key design: `entity:id:version` or include tenant id. Do **not** cache highly sensitive secrets in plain Redis without controls. Keep TTLs short for financial displays or bypass cache for absolute-fresh balances when required.

**Follow-up:** Cache-aside vs write-through? — Cache-aside is most common in Node APIs; simpler and fine for first-round design.

---

### Q4. How do you invalidate cache correctly? `[High]`

**Answer:**
1. On write/update/delete of source data, **delete (or update) related keys**.
2. Prefer short **TTL** as a safety net even with explicit invalidation.
3. Group keys by pattern carefully; avoid `KEYS *` in prod — use versioned keys (`user:42:v7`) and bump version.
4. For list caches, invalidate the list key when any member changes — or don’t cache volatile lists.
5. Be explicit about consistency: users may see stale data for TTL seconds unless you invalidate on write.

**Follow-up:** Stampede after expiry? — Soft TTL / singleflight / random jitter on TTL.

---

### Q5. How do indexes and read replicas help performance? `[Critical]`

**Answer:**
**Indexes** speed `WHERE`/`JOIN`/`ORDER BY` by avoiding full table scans — create them for real query patterns; too many hurt writes. Use `EXPLAIN` to verify. **Read replicas** receive async copies of primary data; route heavy **SELECTs** to replicas, keep **writes** on primary. Good for reports and read-heavy APIs. Accept replica lag for non-critical reads; money-critical reads may need primary.

**Follow-up:** Sequelize read replication? — Configure `replication: { write, read: [...] }` so the library routes queries.

---

### Q6. When do you introduce a message queue? `[High]`

**Answer:**
Use a queue when work is **slow, bursty, or can be async**: emails, SMS, statement generation, webhooks, bulk imports. API enqueues a job and returns quickly; workers process with retries and concurrency limits. Decouples producers from consumers and smooths traffic spikes. For first-round: Redis + Bull/BullMQ or cloud SQS is enough — you don’t need a full event-driven mesh.

**Follow-up:** At-least-once delivery implication? — Design consumers **idempotent**.

---

### Q7. How do you rate-limit an API? `[High]`

**Answer:**
Limit requests per user/IP/API-key in a time window (e.g. 100/min). Implement with Redis counters (INCR + EXPIRE) or a token bucket; put middleware early in Express. Return `429` with `Retry-After`. Stricter limits on login, OTP, and payment endpoints. At edge/WAF for coarse protection; app-level for authenticated quotas. Banking APIs often also throttle per corporate client contract.

**Follow-up:** Distributed rate limit across instances? — Must use shared store (Redis), not in-memory Map.

---

### Q8. What is idempotency and where do you apply it? `[Critical]`

**Answer:**
An idempotent operation can be safely **retried** without changing the result beyond the first success. Critical for payments, fund transfers, and any POST that money touches. Client sends `Idempotency-Key`; server stores key → response; duplicates replay the stored outcome. Also apply on queue consumers so duplicate messages don’t double-process. GET/PUT are naturally closer to idempotent; POST needs explicit design.

**Follow-up:** Unique DB constraint role? — Enforces “process once” even under race of two parallel requests.

---

### Q9. Why connection pooling and how do you size it? `[Critical]`

**Answer:**
Opening a new DB connection per query is expensive. A **pool** keeps reusable connections. In Node, each instance has a pool (Sequelize `pool.max`). Total connections ≈ `instances × pool.max` must stay under DB `max_connections`. Prefer modest pools (e.g. 5–20) plus more instances carefully. Set acquire timeout so requests fail fast when pool is exhausted instead of hanging.

**Follow-up:** Symptom of undersized pool? — Rising acquire wait time, p99 latency spikes, timeouts under load.

---

### Q10. Design a simple notification system (email/SMS/push) for a banking app. `[High]`

**Answer:**
**Flow:**
1. Domain service completes an action (transfer success) and writes an event/outbox row or pushes to a queue (`notification_jobs`).
2. **Notification worker** consumes jobs: pick channel (email/SMS/push), render template, call provider APIs with timeout + retry.
3. Store delivery status (`queued`/`sent`/`failed`) for audit; mask PII in logs.
4. **Preferences** service: user opt-in/opt-out per channel/type — check before send.
5. Providers behind an interface so SES/SMS gateway can swap; circuit-break on provider outages.
6. API stays fast; all sends async except rare OTP paths where you may wait with a strict timeout.

**Follow-up:** How avoid duplicate SMS on retry? — Idempotency key per notification id; provider-side dedupe if available.

---

## Quick “whiteboard” sketch (memorize)

```
Client → LB → Node API (N instances, stateless)
                │
                ├─ Redis (cache, rate limit, sessions, queues)
                ├─ Postgres Primary (writes)
                │      └─ Read Replicas (reads)
                └─ Queue workers → Email/SMS/Push providers
```

Speak this sketch in under 2 minutes, then dive into whichever box they ask about.
