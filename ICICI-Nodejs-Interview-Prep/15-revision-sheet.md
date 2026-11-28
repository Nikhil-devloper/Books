# 15 — Last-Hour Revision Sheet

Dense one-liners. Read top to bottom. Say answers out loud.

---

## Node.js core `[Critical]`

- Node = single JS thread + **event loop** + threadpool (libuv) for some FS/crypto/DNS.
- Never block the loop with heavy CPU sync work — offload to worker threads/queue.
- `process.nextTick` → microtasks → timers → I/O — know “don’t starve the loop.”
- CommonJS `require` vs ESM `import` — project style consistency.
- Streams for large data; don’t `fs.readFile` huge files into memory.
- Cluster/PM2/k8s = multiple processes to use multi-core.
- `unhandledRejection` / `uncaughtException` → log + exit; let orchestrator restart.
- Memory leak suspects: global caches, unbounded Maps, lingering listeners, large closures.

## Express `[Critical]`

- Middleware order matters; errors go to `(err, req, res, next)`.
- Async errors: wrap or use a library — forgotten `try/catch` = hanging/crash.
- Keep controllers thin; services hold business logic; validate input early.
- Central error handler → consistent JSON errors; no stack traces in prod.
- Helmet, CORS allowlist, body size limits, request timeouts.

## Async / Promises `[Critical]`

- `await` in a loop = sequential; use `Promise.all` (with limits) when independent.
- `Promise.all` fails fast; `allSettled` when you need every result.
- Always set **timeouts** on external calls.
- Retry only idempotent ops; exponential backoff + jitter.
- Avoid creating promises you never await (fire-and-forget needs logging + queue).

## Sequelize / SQL `[Critical]`

- Parameterized queries only — never concatenate user input into SQL.
- Transactions for multi-step writes; pass `transaction` everywhere inside.
- N+1 → `include` / `IN (...)` batching; log SQL in staging.
- Indexes match `WHERE`/`JOIN`/`ORDER BY`; prove with `EXPLAIN`.
- Migrations for every schema change; expand-contract for zero-downtime.
- Soft deletes / paranoid — know if your models use them.
- Connection pool: `instances × pool.max` < DB max connections.
- Optimistic lock (`version`) or `SELECT FOR UPDATE` for concurrent updates.

## Security `[Critical]`

- AuthN then AuthZ on every sensitive route.
- JWT: short access TTL, store secrets safely, validate `aud`/`iss` if used.
- Rate-limit login/OTP/payment; lockout / monitoring for abuse.
- Helmet + HTTPS + secure cookies (`httpOnly`, `secure`, `sameSite`).
- No secrets in repo; secret manager; least-privilege DB user.
- Audit log for money-moving actions; mask PII in logs.
- `npm audit` / lockfile; pin dependencies.

## Idempotency & money safety `[Critical]`

- Idempotency-Key + unique constraint = safe retries.
- Exactly-once is hard; aim **at-least-once + idempotent consumer**.
- Double-charge prevention > clever code.
- Outbox pattern when DB commit + message publish must align.

## Caching / Redis `[High]`

- Cache-aside + TTL; invalidate on write.
- Redis also: sessions, rate limits, job queues.
- Don’t cache what must be perfectly fresh unless you have a plan.

## Queues / background jobs `[High]`

- API enqueues → worker sends email/SMS/processes bulk.
- Retries, dead-letter, concurrency caps, idempotent handlers.
- Bull/BullMQ on Redis is a fine interview default.

## Scale & design `[High]`

- Stateless app + LB = horizontal scale.
- Vertical = bigger box; ceiling + SPOF.
- Read replicas for read-heavy; writes to primary; mind lag.
- Rate limit in Redis across instances.
- Health checks: liveness vs readiness (DB up?).

## Jest / testing `[High]`

- Unit: services with mocked DB/clients.
- Integration: real DB in CI when possible; transactional tests.
- Test money edge cases: duplicate key, rollback, 409 conflict.
- `jest.mock` for external APIs; assert retries don’t double-charge.

## Production ops `[High]`

- Structured logs + request/correlation id.
- Metrics: latency p95/p99, error rate, pool wait, queue depth.
- Slow API triage: time each dependency → fix top offender.
- Crash triage: OOM vs exception vs bad probe.
- Safe deploy: migrations, rolling update, quick rollback.

## Scenario triggers (instant recall) `[Critical]`

| Symptom | First move |
|---------|------------|
| API 5s slow | Time DB vs external; `EXPLAIN`; check N+1 |
| Dual update | Version column or `FOR UPDATE` |
| Insert1 ok, insert2 fail | Transaction + rollback |
| Duplicate payment | Idempotency key + unique constraint |
| Vendor down | Timeout + circuit breaker + degrade |
| Big upload | Presigned URL / stream; not RAM |
| 100k rows | Batch + keyset + queue |
| Email slow API | Queue worker |
| DB connect fail | Pool timeouts + health + alert |
| Prod crash | Logs + OOM vs throw; fix + guard |

## Phrases that sound senior (use sparingly)

- “I’d measure before optimizing.”
- “Writes go through a transaction boundary.”
- “Retries need idempotency.”
- “Total pool size across pods matters.”
- “Fail fast with a timeout rather than hang.”
