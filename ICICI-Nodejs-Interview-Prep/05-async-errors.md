# Async Patterns & Error Handling — ICICI Node.js Backend Interview Prep

Senior Node interviews dig into promises, async/await, failure modes, concurrency, retries, and process lifecycle. Be precise about what runs concurrently, how errors propagate, and how services shut down without dropping in-flight work.

---

### Q1. How does Promise chaining work? `[High]`
**Answer:** A Promise represents a future value: pending, fulfilled, or rejected. `.then` registers success handlers and returns a new Promise; `.catch` handles rejections; `.finally` runs cleanup. Returning a value or another Promise chains asynchronous steps without nested callbacks. Unhandled rejections in a chain bubble to the next `.catch`.
**Example:**
```js
fetchAccount(id)
  .then((account) => fetchBalance(account.id))
  .then((balance) => formatINR(balance))
  .catch((err) => logger.error(err))
  .finally(() => metrics.timing("fetch_done"));
```
**Follow-up:** What does `.then` return if you throw inside it? → A rejected Promise; the next `.catch` receives the error.

---

### Q2. Why prefer async/await over raw Promise chains? `[Critical]`
**Answer:** `async/await` is syntactic sugar over Promises that reads like synchronous code and makes `try/catch` natural. An `async` function always returns a Promise. Use `await` for sequential dependent steps; use `Promise.all` when steps are independent. Avoid awaiting inside loops when operations can run in parallel safely.
**Example:**
```js
async function getStatement(accountId) {
  const account = await fetchAccount(accountId);
  const txs = await fetchTransactions(account.id);
  return { account, txs };
}
```
**Follow-up:** Does `await` block the Node event loop? → No. It yields the async function; the event loop continues other work until the Promise settles.

| Style | Pros | Cons |
|---|---|---|
| Callbacks | Historical Node APIs | Pyramid of doom, error-first noise |
| Promise chains | Composable | Can get hard to read |
| async/await | Clearest control flow | Easy to accidentally serialize work |

---

### Q3. How does try/catch interact with async/await? `[Critical]`
**Answer:** `try/catch` around `await` catches that Promise’s rejection. It does not catch errors from floating Promises you did not await, or from callbacks scheduled later (`setTimeout`) unless those are wrapped. In Express 4, wrap async handlers so rejections call `next(err)`.
**Example:**
```js
async function transfer(req, res, next) {
  try {
    const result = await transferService.execute(req.body);
    res.status(201).json(result);
  } catch (err) {
    next(err);
  }
}

// Helper wrapper
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);
```
**Follow-up:** `try/catch` around `fs.readFile` without promisify? → Won’t catch async callback errors; use `fs.promises` or wrap in a Promise.

---

### Q4. What are unhandled Promise rejections and why do they matter? `[Critical]`
**Answer:** An unhandled rejection is a rejected Promise with no `.catch` / surrounding `try/catch` on await. Node emits `unhandledRejection`; modern Node may terminate the process depending on version/flags. Always handle rejections at boundaries: HTTP handlers, queue consumers, and top-level bootstraps. Log and decide whether to crash (fail-fast) for unknown corruption.
**Example:**
```js
process.on("unhandledRejection", (reason) => {
  logger.error({ reason }, "unhandledRejection");
  // In many production services: initiate gracefulShutdown() then exit
});

process.on("uncaughtException", (err) => {
  logger.error({ err }, "uncaughtException");
  // Process state may be unsafe — shut down
});

// Bad — floating promise
sendEmail(user); // if this rejects → unhandledRejection

// Good
void sendEmail(user).catch((err) => logger.error(err));
```
**Follow-up:** `uncaughtException` vs `unhandledRejection`? → Sync throw / EventEmitter errors vs rejected Promises; both are process-level last resorts, not business error handling.

---

### Q5. How do you design custom error classes? `[High]`
**Answer:** Extend `Error`, set `name`, preserve stack (`Error.captureStackTrace` or `super(message)` in modern Node), and add fields like `statusCode`, `code`, and `details`. Controllers/middleware map known classes to HTTP responses; unknown errors become 500. Keep operational errors (expected) separate from programmer errors.
**Example:**
```js
class AppError extends Error {
  constructor(message, { statusCode = 500, code = "APP_ERROR", details } = {}) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.code = code;
    this.details = details;
    this.isOperational = true;
  }
}

class NotFoundError extends AppError {
  constructor(resource = "Resource") {
    super(`${resource} not found`, { statusCode: 404, code: "NOT_FOUND" });
  }
}

class InsufficientFundsError extends AppError {
  constructor() {
    super("Insufficient funds", { statusCode: 409, code: "INSUFFICIENT_FUNDS" });
  }
}

// usage
throw new InsufficientFundsError();
```
**Follow-up:** Why not throw plain strings? → Lose stack traces and structured fields; always throw `Error` instances.

---

### Q6. What are race conditions in Node backends? `[Critical]`
**Answer:** A race condition happens when concurrent requests interleave reads and writes so the final state depends on timing — e.g. two transfers reading the same balance before either writes. Node is single-threaded for JS, but I/O concurrency still creates logical races across awaits and across multiple processes. Fix with DB transactions, row locks (`SELECT FOR UPDATE`), optimistic versioning, or atomic updates.
**Example:**
```js
// Unsafe
const acct = await db.accounts.findById(id);
if (acct.balance >= amount) {
  await db.accounts.update(id, { balance: acct.balance - amount }); // race here
}

// Safer — atomic SQL
await db.query(
  `UPDATE accounts SET balance = balance - $1
   WHERE id = $2 AND balance >= $1`,
  [amount, id]
);
// Check rowCount === 1
```
**Follow-up:** Optimistic locking? → Store `version` column; update only if version matches; retry on conflict.

---

### Q7. Concurrent vs sequential async work? `[Critical]`
**Answer:** Sequential `await` runs one step after another — required when later steps depend on earlier results or when you must limit load. Concurrent patterns (`Promise.all`, `Promise.allSettled`, `Promise.race`) overlap I/O wait time. Use concurrency limits (`p-map`, batches) for external APIs and DB pools so you do not stampede dependencies.
**Example:**
```js
// Sequential — slower, ordered, dependent
const a = await step1();
const b = await step2(a);

// Concurrent — independent I/O
const [profile, ledger, limits] = await Promise.all([
  getProfile(userId),
  getLedger(userId),
  getLimits(userId),
]);

// Partial failure tolerant
const results = await Promise.allSettled(ids.map(fetchAccount));
```
**Follow-up:** `Promise.all` vs `allSettled` vs `race` vs `any`?

| Helper | Behavior |
|---|---|
| `Promise.all` | Fulfills with all results; rejects on first failure |
| `Promise.allSettled` | Waits for all; returns `{status, value/reason}[]` |
| `Promise.race` | Settles with the first settled Promise |
| `Promise.any` | Fulfills with first fulfillment; rejects if all reject (`AggregateError`) |

---

### Q8. How do you handle background processing in Node APIs? `[High]`
**Answer:** Do not run long jobs inside the request/response cycle if the client should get a fast ack. Enqueue work (BullMQ/RabbitMQ/SQS/Kafka), return `202 Accepted` with a job id, and process in workers. For light fire-and-forget tasks, still catch errors and consider process shutdown draining. Keep request handlers idempotent when jobs may retry.
**Example:**
```js
app.post("/reports", authenticate, async (req, res) => {
  const job = await reportQueue.add("statement", {
    userId: req.user.id,
    from: req.body.from,
    to: req.body.to,
  }, { attempts: 3, backoff: { type: "exponential", delay: 2000 } });

  res.status(202).json({ jobId: job.id, statusUrl: `/reports/jobs/${job.id}` });
});
```
**Follow-up:** In-process `setImmediate` for emails? → Risky under load and on crash; a durable queue is preferred for anything financial or customer-visible.

---

### Q9. What is graceful shutdown? `[Critical]`
**Answer:** Graceful shutdown stops accepting new work, finishes in-flight requests/jobs within a deadline, closes DB/queue connections, then exits. Listen for `SIGTERM`/`SIGINT` (containers, k8s, PM2). Force-exit after a timeout so a stuck request cannot hang the process forever. Pair with readiness probes that fail once shutdown starts.
**Example:**
```js
const server = app.listen(3000);
let shuttingDown = false;

async function shutdown(signal) {
  if (shuttingDown) return;
  shuttingDown = true;
  logger.info(`${signal} received`);

  server.close(async () => {
    try {
      await reportQueue.close();
      await db.end();
      process.exit(0);
    } catch (err) {
      logger.error(err);
      process.exit(1);
    }
  });

  setTimeout(() => {
    logger.error("Forced shutdown");
    process.exit(1);
  }, 25_000).unref();
}

process.on("SIGTERM", () => shutdown("SIGTERM"));
process.on("SIGINT", () => shutdown("SIGINT"));

app.use((req, res, next) => {
  if (shuttingDown) return res.status(503).json({ error: "Shutting down" });
  next();
});
```
**Follow-up:** Why `server.close`? → Stops new connections but lets existing keep-alive requests finish; combine with load balancer deregistration.

---

### Q10. How do retry mechanisms work for transient failures? `[High]`
**Answer:** Retry only idempotent or safely-replayable operations on transient errors (timeouts, 429, 503, connection resets) — not on validation or auth failures. Use exponential backoff with jitter to avoid thundering herds. Cap max attempts; log each retry; prefer queue-level retries for background work. Combine with idempotency keys for POSTs.
**Example:**
```js
async function withRetry(fn, { retries = 3, baseMs = 200 } = {}) {
  let lastErr;
  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      return await fn(attempt);
    } catch (err) {
      lastErr = err;
      if (!isTransient(err) || attempt === retries) throw err;
      const delay = baseMs * 2 ** attempt + Math.random() * baseMs;
      await new Promise((r) => setTimeout(r, delay));
    }
  }
  throw lastErr;
}

function isTransient(err) {
  return err.code === "ECONNRESET" || err.statusCode === 429 || err.statusCode === 503;
}
```
**Follow-up:** Why jitter? → Spreads retries so many clients do not retry in lockstep after an outage.

| Strategy | Pattern | Use |
|---|---|---|
| Fixed delay | wait N ms | Simple demos |
| Exponential backoff | N, 2N, 4N… | Most HTTP/DB transients |
| + full jitter | random(0, backoff) | High fan-out clients |
| Circuit breaker | stop calling unhealthy dep | Cascading failure protection |

---

### Q11. How do you implement timeouts correctly? `[High]`
**Answer:** Every outbound call (HTTP, DB, Redis) needs a timeout so hung dependencies cannot exhaust the event loop or connection pool. Prefer AbortController with `fetch`, socket timeouts for DB drivers, and an overall request deadline. On timeout, abort work, free resources, and return `504`/`503` as appropriate — do not leave orphaned Promises ignoring cancelation.
**Example:**
```js
async function fetchWithTimeout(url, ms, options = {}) {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), ms);
  try {
    const res = await fetch(url, { ...options, signal: controller.signal });
    return res;
  } finally {
    clearTimeout(timer);
  }
}

// Promise.race pattern (ensure abort/cleanup still happens)
const result = await Promise.race([
  doWork(),
  new Promise((_, reject) =>
    setTimeout(() => reject(Object.assign(new Error("Timeout"), { code: "ETIMEDOUT" })), 3000)
  ),
]);
```
**Follow-up:** Nested timeouts? → Set service budget (e.g. 3s) less than upstream gateway timeout (e.g. 5–10s) so you fail before the proxy cuts the client.

---

### Q12. How do you avoid losing errors in concurrent helpers? `[Medium]`
**Answer:** `Promise.all` fails fast but other in-flight tasks keep running unless aborted. Track AbortController signals for cancelation. For “all results needed,” use `allSettled` and inspect failures. Never swallow errors with empty `.catch(() => {})` unless you consciously degrade and metric the skip.
**Example:**
```js
const controller = new AbortController();
try {
  await Promise.all([
    callA({ signal: controller.signal }),
    callB({ signal: controller.signal }),
  ]);
} catch (err) {
  controller.abort(); // best-effort cancel siblings
  throw err;
}
```
**Follow-up:** Memory leaks from abandoned Promises? → Long-lived unresolved Promises holding closures can retain memory; timeouts and abort help.

---

### Q13. Sequential map vs parallel map — common interview trap? `[Medium]`
**Answer:** `arr.map(async ...)` returns an array of Promises immediately and does not wait; you need `Promise.all`. `for...of` with `await` is sequential. Accidental parallel `map` can overwhelm a database. Be explicit: sequential, full parallel, or limited concurrency.
**Example:**
```js
// Sequential
for (const id of ids) {
  await processOne(id);
}

// Unbounded parallel
await Promise.all(ids.map((id) => processOne(id)));

// Limited concurrency (simple batching)
const size = 5;
for (let i = 0; i < ids.length; i += size) {
  await Promise.all(ids.slice(i, i + size).map(processOne));
}
```
**Follow-up:** When must it be sequential? → Shared mutable state without locks, rate-limited APIs, or strict ordering (ledger postings).

---

### Q14. How should async errors surface in Express vs workers? `[High]`
**Answer:** In HTTP servers, convert domain errors to HTTP responses via error middleware; log with request id; do not crash on expected operational errors. In queue workers, throw/fail the job so the broker can retry; use dead-letter queues after max attempts; crash only on corrupt process state. Same custom error classes, different boundaries.
**Example:**
```js
// Worker
reportQueue.process(async (job) => {
  try {
    await generateStatement(job.data);
  } catch (err) {
    if (err.isOperational && err.code === "NOT_FOUND") {
      // no retry — acknowledge failure permanently
      return;
    }
    throw err; // retryable
  }
});
```
**Follow-up:** Should validation errors retry? → No — fix the payload; retries would burn the same failure.

---

### Q15. What is backpressure and why does it matter with async streams? `[Medium]`
**Answer:** Backpressure is when a fast producer outruns a slow consumer. In Node streams, `pipe` / `pipeline` handles pause/resume; ignoring it can blow memory. For queues and HTTP uploads, limit concurrency and body size. Prefer `stream.pipeline` with proper error forwarding over manual `pipe` alone.
**Example:**
```js
import { pipeline } from "stream/promises";
import { createReadStream, createWriteStream } from "fs";
import { createGzip } from "zlib";

await pipeline(
  createReadStream("ledger.csv"),
  createGzip(),
  createWriteStream("ledger.csv.gz")
);
```
**Follow-up:** Huge JSON arrays in memory? → Prefer streaming parsers or pagination instead of `JSON.parse` of multi-hundred-MB payloads.
