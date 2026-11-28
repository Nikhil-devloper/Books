# 12 — Practical Scenarios (ICICI Node.js First Round)

Practical answers for a 30-minute first technical round (7+ years). Focus on what you would **do**, not textbook theory.

---

### Q1. An API is taking 5 seconds. How do you find and fix the cause? `[Critical]`

**Answer:**
1. Confirm with logs/APM: is it every request or some? Note `p50` vs `p99`.
2. Check the path: middleware → controller → DB/external calls. Log timings per step (or use New Relic / Datadog / OpenTelemetry).
3. Common causes: missing index, N+1 queries, sequential `await` that could run in parallel, slow third-party API, large payload serialization.
4. Fix the top bottleneck first (usually DB or external HTTP). Add timeout + caching if external. Re-measure before and after.
5. If still slow under load: check connection pool exhaustion and event-loop blocking (CPU-heavy sync work).

**Follow-up:** How would you prove the DB was the bottleneck? — Explain query timing logs / `EXPLAIN ANALYZE` / slow-query log.

---

### Q2. Two users update the same record at the same time. How do you handle it? `[Critical]`

**Answer:**
1. Prefer **optimistic locking**: add a `version` (or `updated_at`) column; `UPDATE ... WHERE id=? AND version=?`; if 0 rows updated → conflict → return 409 and ask client to refresh.
2. Or use a **DB transaction** with `SELECT ... FOR UPDATE` when the business must serialize (balance deduction, inventory).
3. Keep the lock window short: do validation outside the lock; only mutate inside.
4. Never rely on “read → change in JS → write” without a version or lock — last write wins and corrupts data.
5. Return a clear error so the UI can retry with fresh data.

**Follow-up:** Optimistic vs pessimistic — when each? — High contention / money → pessimistic or atomic SQL; low contention → optimistic.

---

### Q3. First DB insert succeeds, second insert fails. What do you do? `[Critical]`

**Answer:**
1. Wrap both inserts in a **single transaction** (`BEGIN` → insert1 → insert2 → `COMMIT`). On any failure → `ROLLBACK`.
2. In Sequelize: `sequelize.transaction(async (t) => { await Model.create(..., { transaction: t }); ... })`.
3. Make sure you pass `transaction: t` to **every** query inside the block.
4. Catch errors, log with request id, return a safe 5xx/4xx — never leave orphan parent rows.
5. If the second step is an external API (not DB), use outbox / saga / compensating action — a DB transaction alone cannot undo an HTTP call.

**Follow-up:** What if the process crashes after commit of step 1 but before step 2 outside a transaction? — Idempotent retry + reconciliation job.

---

### Q4. How do you prevent duplicate payment / double charge? `[Critical]`

**Answer:**
1. Client sends a unique **idempotency key** (UUID) with each payment attempt.
2. Store the key in DB with a **unique constraint**. First request processes; duplicate key → return the same stored response (do not charge again).
3. Process payment inside a transaction: insert payment row (unique key) + call gateway only after lock/insert succeeds.
4. Prefer gateway’s own idempotency support if available; still keep your own record.
5. Make the payment worker **idempotent**: safe to retry; status machine (`pending` → `success`/`failed`) with clear transitions.

**Follow-up:** What if the gateway charged but your DB write failed? — Query gateway by reference id; reconcile; never blindly retry charge without checking status.

---

### Q5. An external API your service depends on is unavailable. How do you design for this? `[Critical]`

**Answer:**
1. Set **timeouts** on every outbound call (never hang forever).
2. Use **retries with exponential backoff + jitter** only for safe/idempotent reads or when the provider supports idempotency.
3. **Circuit breaker**: after N failures, fail fast for a cool-down period; poll health; half-open then close.
4. Degrade gracefully: cached last-good data, queue for later, or clear user message — depending on business need.
5. Log/metrics/alerts on error rate and latency; isolate the integration in a client module so the rest of the API stays healthy.

**Follow-up:** Retry on POST payment? — Usually no, unless idempotency key is guaranteed.

---

### Q6. User uploads a large file (e.g. 100MB+). How do you handle it? `[High]`

**Answer:**
1. Do **not** buffer the whole file in memory. Use streaming (`busboy` / `multer` disk/stream) or direct-to-object-storage (**presigned URL** to S3/GCS) — preferred.
2. Validate size, MIME/type, and auth **before** accepting. Reject early.
3. Store file metadata in DB; store blob in object storage. Virus scan async if required (banking context).
4. Process (resize, parse CSV) in a **background worker**, not in the request thread.
5. Set reverse-proxy / body size limits so abuse cannot take down Node.

**Follow-up:** Why presigned upload? — Offloads bandwidth from Node; better scale and security boundary.

---

### Q7. You need to process 100,000 records. How? `[High]`

**Answer:**
1. Do **not** `findAll()` everything into memory. Stream or paginate with keyset (`WHERE id > lastId ORDER BY id LIMIT 1000`).
2. Process in batches; control concurrency (e.g. `p-limit`) so you do not exhaust DB pool or CPU.
3. Prefer a **job queue** (Bull/BullMQ + Redis): enqueue batches, workers process, track progress/failures.
4. Make each batch **idempotent** and restartable; persist checkpoint (`last_processed_id`).
5. Report status via job id / admin API; alert on failure rate.

**Follow-up:** Why keyset over `OFFSET`? — `OFFSET` gets slower as offset grows; keyset stays O(log n) with index.

---

### Q8. How do you send email in the background without slowing the API? `[High]`

**Answer:**
1. API writes the business change + an **outbox/queue message**, then returns 201/200 quickly.
2. Worker (Bull/BullMQ, SQS, etc.) picks the job and sends email via SMTP/SES.
3. Persist job status; retry failed sends with backoff; dead-letter after max attempts.
4. Never `await transporter.sendMail()` on the critical request path for user-facing latency.
5. Include template id + data in the job payload; avoid huge payloads — store refs if needed.

**Follow-up:** What if email job is lost? — Transactional outbox: same DB transaction as business write, then publisher drains outbox.

---

### Q9. Database connection fails at runtime. What happens and how do you harden it? `[Critical]`

**Answer:**
1. Pool should surface errors; health check (`/health`) should verify DB connectivity for k8s/LB.
2. Configure pool size, acquire timeout, idle timeout; log pool exhaustion separately from query errors.
3. Transient failures: retry only safe reads with backoff; fail write requests clearly (503) rather than hang.
4. Sequelize/pg: handle `SequelizeConnectionError`; do not leave hanging requests — always timeout.
5. Ops side: multi-AZ DB, automatic failover, connection string to primary; alert on connection error rate.

**Follow-up:** Why can too many Node instances hurt the DB? — Each multiplies pool size → too many connections → DB meltdown.

---

### Q10. What is N+1 and how do you fix it in Sequelize/SQL? `[Critical]`

**Answer:**
1. N+1 = 1 query for list + N queries for related rows (e.g. users then each user’s accounts).
2. Fix with **eager loading**: `include` / `JOIN`, or one `WHERE id IN (...)` for related ids.
3. For lists, select only needed columns; avoid nested includes that explode rows.
4. Detect with logging: enable Sequelize SQL log in staging; look for repeated similar queries in a loop.
5. For GraphQL-like patterns, use DataLoader-style batching.

**Follow-up:** Show a bad vs good pattern — loop `await Account.findAll({ where: { userId }})` vs single `findAll` with `userId: ids` or `include`.

---

### Q11. Production Node process crashed / restarted. How do you investigate? `[Critical]`

**Answer:**
1. Check process manager / k8s events: OOMKilled vs uncaught exception vs health-check kill.
2. Pull last logs before exit; search for unhandledRejection / uncaughtException stack.
3. If OOM: heap snapshot / memory trend; look for unbounded arrays, caches without TTL, large buffers.
4. If exception: fix root cause; ensure global handlers log and exit cleanly; use PM2/k8s restart policy.
5. Add metrics: restart count, event-loop lag, heap used; reproduce with same traffic pattern in staging.

**Follow-up:** unhandledRejection policy? — Log with context and exit so the process does not run half-broken; let orchestrator restart.

---

### Q12. How do you secure a Node.js API (banking-relevant basics)? `[Critical]`

**Answer:**
1. **AuthN/AuthZ**: JWT/session with short TTL + refresh; authorize every route (role/permission); never trust client ids alone.
2. **Transport & headers**: HTTPS only; Helmet; CORS allowlist; disable verbose errors in prod.
3. **Input**: validate/sanitize with schema (Joi/Zod); parameterized queries only (Sequelize bindings) — no string-built SQL.
4. **Secrets**: env/secret manager; never commit keys; rotate; least privilege DB user.
5. **Abuse**: rate limit, idempotency on money APIs, audit log for sensitive actions, dependabot/npm audit for CVEs.

**Follow-up:** OWASP Top risks you watch? — Injection, broken auth, SSRF to internal services, sensitive data exposure.

---

### Q13. How do you scale a Node.js service? `[High]`

**Answer:**
1. **Horizontal**: multiple instances behind a load balancer; keep app **stateless** (session in Redis, files in S3).
2. Use Node **cluster** or k8s pods to use multiple CPU cores (one event loop per process).
3. Offload heavy/slow work to **queues + workers**; cache hot reads in Redis.
4. Tune DB: indexes, read replicas for read-heavy traffic, connection pooling sized to instance count.
5. Observe first: CPU, event-loop lag, p99 latency, error rate — scale on signals, not guesswork.

**Follow-up:** Sticky sessions? — Avoid if possible; prefer shared session store so any instance can serve any user.
