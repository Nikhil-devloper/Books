# 09 — Performance & Scalability (Node.js Backend)

First technical round focus: making APIs fast, stable, and ready to scale.

---

### Q1. How do you optimize a slow Node.js API? `[Critical]`
**Answer:** Measure first: find slow endpoints with logs, APM, or simple timing middleware. Typical wins are adding indexes, caching hot reads, reducing over-fetching (select only needed columns), and avoiding N+1 queries. Move heavy work (PDF, reports, emails) to background jobs so the request returns quickly. Keep the event loop free — no long CPU-bound sync work on the main thread.
**Example:**
```js
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    console.log(`${req.method} ${req.path} ${Date.now() - start}ms`);
  });
  next();
});
```
**Follow-up:** What would you check first — DB query time or Node CPU?

---

### Q2. How does Redis help API performance? `[Critical]`
**Answer:** Redis is an in-memory store used for caching, session data, rate-limit counters, and short-lived locks. Reading from memory is much faster than hitting the database every time. Use it for data that is read often and changes less often (config, product lists, user profile snapshots). Always set TTLs so stale keys expire automatically.
**Example:**
```js
async function getUser(id) {
  const key = `user:${id}`;
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  const user = await db.users.findById(id);
  await redis.set(key, JSON.stringify(user), 'EX', 300); // 5 min
  return user;
}
```
**Follow-up:** What kinds of data should you *not* cache in Redis?

---

### Q3. How do you invalidate cache correctly? `[High]`
**Answer:** Invalidation means removing or updating cache when the source data changes. Common patterns: TTL expiry, delete key on write (`DEL user:123`), or versioned keys (`user:123:v5`). For related data, use key prefixes/tags carefully so one update can clear a group. Wrong invalidation causes stale reads — often worse than a slightly slower correct read.
**Example:**
```js
async function updateUser(id, data) {
  const user = await db.users.update(id, data);
  await redis.del(`user:${id}`); // write-through invalidation
  return user;
}
```
**Follow-up:** TTL-only vs explicit delete — when is each better?

---

### Q4. Horizontal vs vertical scaling — what is the difference? `[Critical]`
**Answer:** Vertical scaling means a bigger machine (more CPU/RAM). Horizontal scaling means more machines behind a load balancer. Vertical is simpler but hits a hardware ceiling and has a single point of failure. Horizontal scales further and improves availability, but needs a stateless app design and shared storage (DB/Redis). In cloud systems you usually grow horizontally after basic vertical sizing.
**Example:**
```text
Vertical:  1 server 2 CPU → 1 server 8 CPU
Horizontal: 1 server → 4 identical app servers + load balancer
```
**Follow-up:** Why must sessions live outside the Node process for horizontal scaling?

---

### Q5. What is load balancing and why do we use it? `[High]`
**Answer:** A load balancer distributes incoming traffic across multiple app instances. It improves throughput and keeps the service up if one instance dies (health checks remove bad nodes). Common algorithms are round-robin, least connections, and IP hash. For Node APIs, put Nginx, HAProxy, or a cloud LB (ALB/NLB) in front of several processes or containers.
**Example:**
```text
Client → Load Balancer → App1
                       → App2
                       → App3
```
**Follow-up:** What should a `/health` endpoint check?

---

### Q6. What are read replicas and when do you use them? `[High]`
**Answer:** A read replica is a copy of the primary database that serves read queries. Writes still go to the primary; reads can be split to replicas to reduce primary load. Useful for reporting, listings, and heavy GET traffic. Be aware of replication lag — a read right after a write might not see the new data yet on a replica.
**Example:**
```js
async function getOrders(userId) {
  // read from replica pool
  return readDb.query('SELECT * FROM orders WHERE user_id = $1', [userId]);
}

async function createOrder(order) {
  // write to primary
  return writeDb.query('INSERT INTO orders ...', [...]);
}
```
**Follow-up:** How would you handle “read-your-writes” consistency?

---

### Q7. What is connection pooling and why does it matter? `[High]`
**Answer:** Opening a new DB connection per request is slow and can exhaust database limits. A pool keeps a set of reusable connections ready for the app. Size the pool based on instance count × pool size ≤ database max connections. In Node, libraries like `pg` Pool or ORM pools handle checkout/release for you.
**Example:**
```js
const { Pool } = require('pg');
const pool = new Pool({
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 5000,
});

const { rows } = await pool.query('SELECT 1');
```
**Follow-up:** What happens if pool size is too large across many app pods?

---

### Q8. How do message queues help performance? `[High]`
**Answer:** Queues (RabbitMQ, SQS, Bull/BullMQ with Redis) move slow or retryable work off the request path. The API enqueues a job and returns quickly; workers process emails, SMS, statements, or batch updates asynchronously. Queues also smooth traffic spikes and support retries with dead-letter queues for failures.
**Example:**
```js
// API process
await emailQueue.add('welcome', { userId: user.id });

// Worker process
emailQueue.process('welcome', async (job) => {
  await sendWelcomeEmail(job.data.userId);
});
```
**Follow-up:** At-least-once delivery — why must consumers be idempotent?

---

### Q9. What causes memory leaks in Node.js and how do you find them? `[High]`
**Answer:** Common causes: unbounded arrays/maps, global caches without TTL, event listeners never removed, and closures holding large objects. Symptoms are rising RSS/heap over time and eventual OOM restarts. Use `clinic`, Chrome DevTools heap snapshots, or `node --inspect` to compare heaps. Fix by bounding caches, clearing timers/listeners, and avoiding accidental global state growth.
**Example:**
```js
// BAD — grows forever
const cache = {};
function remember(key, value) {
  cache[key] = value;
}

// BETTER — bounded LRU or Redis with TTL
```
**Follow-up:** How is a memory leak different from high but stable memory use?

---

### Q10. How does garbage collection affect Node performance? `[Medium]`
**Answer:** V8 GC automatically frees unused heap memory. Short pauses are normal; long GC pauses can delay request handling. Large heaps and many short-lived objects increase GC work. Avoid creating huge temporary objects in hot paths and do not hold references longer than needed. Monitor GC metrics in production when latency spikes appear.
**Example:**
```bash
node --max-old-space-size=2048 app.js
# Use APM or --trace_gc only while debugging, not permanently in prod logs
```
**Follow-up:** What is the difference between young-space GC and old-space GC at a high level?

---

### Q11. Why should Node.js apps be stateless for scaling? `[Critical]`
**Answer:** A stateless app does not keep must-have session data only in local memory. Any instance can handle any request, which makes horizontal scaling and rolling deploys easy. Store sessions in Redis, files in S3, and durable state in the database. Local memory is fine for short caches, but not as the only source of truth.
**Example:**
```text
Bad:  login stores session Map in process memory → sticky sessions required
Good: login stores session in Redis → any pod can validate the session
```
**Follow-up:** Are sticky sessions a good long-term design?

---

### Q12. How do you design background jobs well? `[High]`
**Answer:** Put non-urgent or heavy work in jobs: emails, PDF generation, reconciliation, image processing. Make jobs idempotent, set retries with backoff, and use a dead-letter queue for poison messages. Record job status for user-facing progress when needed. Keep payloads small (IDs, not whole documents).
**Example:**
```js
await reportQueue.add(
  'monthly-statement',
  { accountId },
  { attempts: 3, backoff: { type: 'exponential', delay: 2000 } }
);
```
**Follow-up:** Sync API vs async job — how do you choose?

---

### Q13. How can you reduce database load without adding servers? `[High]`
**Answer:** Add the right indexes, avoid `SELECT *`, paginate large lists, and batch writes when possible. Cache frequent reads in Redis. Denormalize carefully for hot read paths. Fix N+1 queries with joins or DataLoader-style batching. Often query design beats throwing hardware at the problem.
**Example:**
```sql
-- BAD
SELECT * FROM transactions WHERE user_id = 10;

-- BETTER
SELECT id, amount, created_at
FROM transactions
WHERE user_id = 10
ORDER BY created_at DESC
LIMIT 20 OFFSET 0;
-- + index on (user_id, created_at DESC)
```
**Follow-up:** When does an index hurt write performance?

---

### Q14. What is the Node.js cluster module / multi-core usage? `[Medium]`
**Answer:** One Node process uses one CPU core for JS. To use more cores on one machine, run multiple processes (cluster module, PM2 cluster mode, or multiple containers). Each worker is a separate process sharing the port via a master or via the OS/load balancer. This is vertical machine usage via horizontal processes.
**Example:**
```js
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  os.cpus().forEach(() => cluster.fork());
} else {
  require('./server');
}
```
**Follow-up:** Why do people prefer containers + LB over cluster in many cloud setups?

---

### Q15. How do you handle slow third-party APIs? `[High]`
**Answer:** Set strict timeouts, use circuit breakers, and retry only idempotent calls with backoff. Cache third-party responses when freshness allows. Prefer async webhooks or queues over blocking the user request on a slow partner. Fail gracefully with a clear fallback message.
**Example:**
```js
const controller = new AbortController();
const t = setTimeout(() => controller.abort(), 3000);

try {
  const res = await fetch(url, { signal: controller.signal });
  return res.json();
} finally {
  clearTimeout(t);
}
```
**Follow-up:** Why can unlimited retries make an outage worse?

---

### Q16. What is pagination and why does it matter for performance? `[Medium]`
**Answer:** Pagination returns data in pages instead of one huge list. It cuts memory use, response size, and DB load. Offset/limit is simple; cursor-based pagination scales better for large, changing datasets. Always enforce a max page size on the server.
**Example:**
```js
const limit = Math.min(Number(req.query.limit) || 20, 100);
const offset = Number(req.query.offset) || 0;
const rows = await db.query(
  'SELECT id, name FROM items ORDER BY id LIMIT $1 OFFSET $2',
  [limit, offset]
);
```
**Follow-up:** What is the downside of deep `OFFSET` values?

---

### Q17. How do compression and payload size affect API speed? `[Medium]`
**Answer:** Smaller JSON responses transfer faster, especially on mobile networks. Enable gzip/brotli for text responses and avoid sending unused fields. Do not over-compress already-compressed binaries. Also watch request body size limits to protect memory.
**Example:**
```js
const compression = require('compression');
app.use(compression());
app.use(express.json({ limit: '100kb' }));
```
**Follow-up:** When should you not compress responses?

---

### Q18. How do you know if your API is “fast enough”? `[Medium]`
**Answer:** Define SLOs: for example p95 latency under 300ms for critical GETs. Track throughput (RPS), error rate, and saturation (CPU, DB connections, queue depth). Load-test realistic scenarios before big releases. Optimize based on data, not guesses.
**Example:**
```text
Metrics to watch:
- p50 / p95 / p99 latency
- requests per second
- 5xx rate
- DB pool wait time
- Redis hit ratio
```
**Follow-up:** Why is p99 often more important than average latency?

---

## Quick Cheat Sheet

| Lever | Use when | Watch out |
|-------|----------|-----------|
| Redis cache | Hot reads | Stale data / invalidation |
| Horizontal scale | Traffic growth | Must be mostly stateless |
| Read replicas | Read-heavy DB | Replication lag |
| Connection pool | Every DB app | Pool × pods ≤ DB max |
| Message queue | Slow / retry work | Idempotency |
| Background jobs | Non-request-critical | Monitoring & DLQ |
