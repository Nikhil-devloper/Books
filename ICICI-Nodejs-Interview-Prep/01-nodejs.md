# Node.js Interview Prep — ICICI Backend Developer (First Round, 30 min)

**Priority note:** Focus first on Event Loop phases, blocking vs non-blocking, Streams, Cluster vs Worker Threads, and memory leaks — these appear most often in a 30-min senior round. Architecture/V8/libuv and EventEmitter are common warm-ups; output-based Event Loop questions are high-signal for 7+ years.

---

### Q1. What is Node.js architecture? How do V8 and libuv fit in? `[Critical]`
**Answer:** Node.js is a JavaScript runtime built on **Chrome V8** (executes JS, manages heap/GC) and **libuv** (cross-platform async I/O, thread pool, event loop). A single main thread runs JS; libuv offloads file I/O, DNS, crypto, zlib to a **worker thread pool** (default 4). Network I/O uses OS mechanisms (epoll/kqueue/IOCP), not always the thread pool.
**Example:**
```
JS Code → V8 Engine → Node.js Bindings → libuv → OS / Thread Pool
                ↓
           Event Loop (main thread)
```
**Follow-up:** Can you change the thread pool size? → Yes, via `UV_THREADPOOL_SIZE` (1–1024). Useful for heavy fs/crypto workloads.

---

### Q2. Explain the Node.js Event Loop and its phases. `[Critical]`
**Answer:** The Event Loop processes callbacks in ordered phases: **timers** → **pending callbacks** → **idle/prepare** → **poll** → **check** → **close callbacks**. Between phases, **microtasks** (`process.nextTick`, then Promise jobs) run until the queue is empty. `setTimeout`/`setInterval` run in timers; `setImmediate` in check; I/O callbacks mostly in poll.
**Example:**
```
┌───────────────────────────┐
│   timers (setTimeout)     │
│   pending callbacks       │
│   idle, prepare           │
│   poll (I/O)              │
│   check (setImmediate)    │
│   close callbacks         │
└───────────────────────────┘
  ↑ nextTick + Promise microtasks between every phase
```
**Follow-up:** What runs between phases? → `process.nextTick` queue first, then Promise microtask queue, both drained completely before the next phase.

---

### Q3. Node is single-threaded — how does it handle concurrency? `[Critical]`
**Answer:** The **JS main thread is single-threaded**, so only one JS stack runs at a time. Concurrency comes from the Event Loop + non-blocking I/O: while waiting on network/disk, Node accepts other requests. CPU-bound work still blocks the event loop unless moved to **Worker Threads**, **child_process**, or **Cluster**.
**Example:** 10,000 concurrent HTTP clients can be served on one process because most time is waiting on I/O, not computing.
**Follow-up:** Is libuv single-threaded? → No. The event loop runs on the main thread; libuv’s thread pool runs blocking syscalls in parallel.

---

### Q4. Blocking vs non-blocking, sync vs async — how do they differ? `[Critical]`

| Concept | Meaning | Example |
|---------|---------|---------|
| **Blocking** | Main thread waits; event loop stuck | `fs.readFileSync`, tight CPU loop |
| **Non-blocking** | Call returns immediately; work continues elsewhere | `fs.readFile`, `http.get` |
| **Sync** | Result available before next line | `JSON.parse`, `readFileSync` |
| **Async** | Result via callback/Promise later | `readFile`, `fetch` |

**Answer:** Sync ≠ always blocking I/O (CPU sync work is still fine if short). Async APIs can still block if the callback does heavy sync work. Prefer async I/O in servers; use sync only at startup (e.g., loading config).
**Example:**
```js
// Blocking — freezes event loop
const data = fs.readFileSync('big.txt');

// Non-blocking — event loop stays free
fs.readFile('big.txt', (err, data) => { /* ... */ });
```
**Follow-up:** Is `crypto.pbkdf2Sync` bad in a request handler? → Yes — CPU + sync; use `pbkdf2` async or Worker Threads.

---

### Q5. What is `process.nextTick` vs Promise microtasks? `[Critical]`
**Answer:** `process.nextTick` callbacks run **before** Promise microtasks, and both run **before** the next Event Loop phase. NextTick can **starve** the event loop if it recursively schedules itself. Prefer Promises/`queueMicrotask` for most deferred work; reserve `nextTick` for APIs that must emit before the caller continues.
**Example:**
```js
Promise.resolve().then(() => console.log('promise'));
process.nextTick(() => console.log('nextTick'));
// Output: nextTick → promise
```
**Follow-up:** Why does Node prefer nextTick first? → Historical API contract: allow cleanup/emit before returning to user code, before Promises existed.

---

### Q6. `setTimeout(fn, 0)` vs `setImmediate(fn)` — which runs first? `[High]`
**Answer:** Inside an **I/O callback**, `setImmediate` usually runs before `setTimeout(fn, 0)`. Outside I/O (main module), order is **non-deterministic** depending on process performance. `setImmediate` is designed for “after poll”; timers depend on the timers phase and clock.
**Example:**
```js
const fs = require('fs');
fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
});
// Typical: immediate → timeout
```
**Follow-up:** When would you use setImmediate? → Defer heavy work until after current I/O callbacks, without timer delay jitter.

---

### Q7. Explain Streams and when to use them. `[Critical]`
**Answer:** Streams process data in chunks instead of loading everything into memory — critical for large files, HTTP bodies, and pipes. Types: **Readable**, **Writable**, **Duplex**, **Transform**. Prefer `pipeline()` / `stream/promises.pipeline` over `.pipe()` for proper error propagation and cleanup.
**Example:**
```js
const { pipeline } = require('stream/promises');
const fs = require('fs');
const zlib = require('zlib');

await pipeline(
  fs.createReadStream('big.log'),
  zlib.createGzip(),
  fs.createWriteStream('big.log.gz')
);
```
**Follow-up:** What is backpressure? → When writable is slower than readable; streams pause/resume via `highWaterMark` and `drain` so memory doesn’t blow up.

---

### Q8. What are Buffers and how do they relate to Streams? `[High]`
**Answer:** A **Buffer** is a fixed-size chunk of raw binary data outside the V8 heap string model (backed by ArrayBuffer). Streams emit/consume Buffers (or strings if encoding set). Always set encoding or handle binary carefully to avoid corruption; prefer `Buffer.alloc` over deprecated `new Buffer()`.
**Example:**
```js
const buf = Buffer.from('hello', 'utf8');
console.log(buf.length);        // 5
console.log(buf.toString('hex'));
Buffer.alloc(10);               // zero-filled, safe
Buffer.allocUnsafe(10);         // faster, may contain old memory
```
**Follow-up:** Buffer vs TypedArray? → Buffer is a Uint8Array subclass with Node-specific helpers; shares ArrayBuffer under the hood.

---

### Q9. How does EventEmitter work? Common pitfalls? `[High]`
**Answer:** EventEmitter is the Observer pattern: `on`/`once` register listeners; `emit` invokes them synchronously in registration order. Errors on `'error'` with no listener **crash the process**. Default max listeners is 10 (leak warning). Prefer `once` for one-shot events; remove listeners to avoid leaks.
**Example:**
```js
const { EventEmitter } = require('events');
const ee = new EventEmitter();

ee.on('data', (n) => console.log(n));
ee.emit('data', 42); // sync — prints immediately

ee.on('error', (err) => console.error(err)); // required to avoid crash
```
**Follow-up:** Are listeners async? → No — `emit` is sync. Async work inside a listener does not delay subsequent listeners unless awaited by you.

---

### Q10. Cluster vs Worker Threads — when to use which? `[Critical]`

| | **Cluster** | **Worker Threads** |
|---|-------------|-------------------|
| **Purpose** | Scale across CPU cores for network servers | Parallel CPU work inside one process |
| **Memory** | Separate processes (isolated heaps) | Shared process; optional SharedArrayBuffer |
| **IPC** | `process.send` / handles | `postMessage` / MessageChannel |
| **Best for** | HTTP/TCP load on multi-core | Image processing, crypto, parsing |
| **Failure** | One worker crash doesn’t kill others | Uncaught error can affect design; isolate carefully |

**Answer:** Use **Cluster** (or PM2/K8s replicas) to utilize multiple cores for I/O servers. Use **Worker Threads** for CPU-bound tasks without forking a full process. Don’t use Cluster for sharing in-memory state — use Redis/DB.
**Example:**
```js
// Worker Threads sketch
const { Worker } = require('worker_threads');
new Worker('./heavy.js').on('message', console.log);
```
**Follow-up:** Cluster vs multiple Docker replicas? → Similar horizontal scale; orchestration prefers containers over Node cluster in many production setups.

---

### Q11. How do you handle CPU-intensive operations in Node? `[Critical]`
**Answer:** Never run long CPU loops on the main thread — they block all requests. Offload to **Worker Threads**, **child_process**, native addons, or external services. For lighter work, break into chunks with `setImmediate` to yield to the event loop. Monitor event-loop lag (e.g., `perf_hooks.monitorEventLoopDelay`).
**Example:**
```js
// Bad: blocks event loop
function fib(n) { return n < 2 ? n : fib(n-1) + fib(n-2); }

// Better: worker_threads or chunked work
function chunked(arr, i = 0) {
  const end = Math.min(i + 1000, arr.length);
  for (; i < end; i++) processItem(arr[i]);
  if (i < arr.length) setImmediate(() => chunked(arr, i));
}
```
**Follow-up:** Can async/await fix CPU blocking? → No. `await` only yields on async I/O/Promises; sync CPU still blocks until it finishes.

---

### Q12. What causes memory leaks in Node and how does GC work? `[High]`
**Answer:** V8 uses generational GC (young/old space): short-lived objects die in Scavenge; long-lived go to Mark-Sweep/Compact. Leaks come from growing caches, uncleared timers/listeners, global maps, closures holding large objects, and uncleared streams. Detect with `process.memoryUsage()`, heap snapshots, `--inspect`, and clinic.js/heapdump.
**Example:**
```js
// Leak pattern: unbounded cache
const cache = new Map();
app.get('/x', (req, res) => {
  cache.set(req.url + Date.now(), hugeObject); // never deleted
});

// Fix: LRU with max size / TTL (e.g., lru-cache)
```
**Follow-up:** What is retained size vs shallow size? → Shallow is the object alone; retained is memory freed if that object is GC’d (includes what it uniquely references).

---

### Q13. What is the difference between `require` and ES modules `import`? `[Medium]`
**Answer:** `require` is CommonJS — sync, runtime, caches in `require.cache`. ES `import` is static (hoisted), can be async for dynamic `import()`, and uses a separate module map. Mixing needs care (`.mjs`/`.cjs`/`"type": "module"`). CJS is still common in older Node codebases.
**Example:**
```js
// CJS
const fs = require('fs');

// ESM
import fs from 'fs';
const mod = await import('./lazy.js'); // dynamic
```
**Follow-up:** Is require cached? → Yes — subsequent requires return the same exports object (singleton per resolved path).

---

## Output-based Event Loop Questions (expect exact output)

### Q14. What is the output? `[Critical]`
```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
process.nextTick(() => console.log('4'));
console.log('5');
```
**Answer:** Output is **`1 5 4 3 2`**. Sync first (`1`, `5`), then nextTick (`4`), then Promise microtask (`3`), then timers phase (`2`).
**Example:** Order rule: sync → nextTick → Promises → timers/I/O/check.
**Follow-up:** If two nextTicks and two then()? → All nextTicks drain first (FIFO), then all Promise jobs (FIFO), including ones scheduled during drain.

---

### Q15. What is the output? `[Critical]`
```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
```
**Answer:** In the **main module**, order is **not guaranteed** — either `timeout` then `immediate`, or the reverse, depending on event-loop entry timing. Do not rely on this order in production code.
**Example:** To make order deterministic, nest them inside an I/O callback (see Q6) where `setImmediate` typically wins.
**Follow-up:** Why non-deterministic? → Timers phase checks expired timers; if 0ms hasn’t “expired” yet when poll ends, check (immediate) may run first.

---

### Q16. What is the output? `[Critical]`
```js
const fs = require('fs');

fs.readFile(__filename, () => {
  console.log('I/O');
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
  process.nextTick(() => console.log('nextTick'));
  Promise.resolve().then(() => console.log('promise'));
});
```
**Answer:** Output: **`I/O` → `nextTick` → `promise` → `immediate` → `timeout`**. After the I/O callback, microtasks run; then check phase (`setImmediate`) before timers in this common pattern.
**Example:** Microtasks always flush before moving to the next phase after the poll callback returns.
**Follow-up:** Where does `close` callback fit? → Close phase, after check — e.g., `socket.on('close')`.

---

### Q17. What is the output? `[High]`
```js
process.nextTick(() => {
  console.log('nextTick 1');
  process.nextTick(() => console.log('nextTick 2'));
});
Promise.resolve().then(() => {
  console.log('promise 1');
  Promise.resolve().then(() => console.log('promise 2'));
});
console.log('sync');
```
**Answer:** Output: **`sync` → `nextTick 1` → `nextTick 2` → `promise 1` → `promise 2`**. NextTick queue drains completely (including nested) before Promise microtasks start; then Promise queue drains including nested.
**Example:** Nested nextTick before any Promise is why recursive nextTick can starve I/O.
**Follow-up:** How to avoid starvation? → Prefer `queueMicrotask`/Promises or `setImmediate` for recursive deferral that must yield to I/O.

---

### Q18. What is the output? `[High]`
```js
console.log('A');

setTimeout(() => {
  console.log('B');
  Promise.resolve().then(() => console.log('C'));
}, 0);

Promise.resolve().then(() => {
  console.log('D');
  setTimeout(() => console.log('E'), 0);
});

console.log('F');
```
**Answer:** Output: **`A F D B C E`**. Sync `A F`; microtask `D` schedules timer `E`; then timer `B` runs and schedules microtask `C` (runs before next timer); then `E`.
**Example:** Microtasks scheduled inside a timer callback run **before** the next timer callback.
**Follow-up:** Same pattern with nextTick inside setTimeout? → nextTick still runs before Promises after that timer callback returns.

---

### Q19. How would you debug a slow Node.js API in production? `[High]`
**Answer:** Check event-loop lag, CPU vs I/O wait, memory growth, and slow downstream calls. Use APM (latency breakdown), logs with correlation IDs, `clinic doctor`/`flame`, and metrics (p99, GC pauses). Distinguish blocked loop (CPU/sync) from slow await (DB/network).
**Example:** High CPU + rising lag → CPU bound or sync I/O. Low CPU + high latency → waiting on DB/HTTP; optimize queries or concurrency limits.
**Follow-up:** What is event loop utilization? → Fraction of time the loop is busy; ELU near 1.0 means little idle capacity.

---

### Q20. `Buffer` vs `String` for network protocols — what matters? `[Medium]`
**Answer:** Protocols are binary — keep data as Buffer until you need text. Unnecessary `toString()`/`Buffer.from()` copies cost CPU and memory. Set `highWaterMark` thoughtfully on streams for throughput vs memory.
**Example:** Parsing length-prefixed frames: accumulate Buffers, slice by length, avoid concatenating huge strings.
**Follow-up:** What does `Buffer.concat` cost? → Allocates a new Buffer and copies — batch carefully under backpressure.

---

## Quick revision checklist (30-min round)

1. Draw Event Loop phases + where nextTick/Promises fit  
2. Blocking sync I/O vs async I/O examples  
3. Streams + backpressure + `pipeline`  
4. Cluster vs Worker Threads table  
5. One memory-leak pattern + how you’d detect it  
6. Solve 2–3 output-based Event Loop questions out loud  

---

*Priorities: `[Critical]` = very likely · `[High]` = common · `[Medium]` = if time remains*
`)
