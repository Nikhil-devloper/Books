# ICICI Node.js Backend Developer — Interview Prep

First **technical round** prep pack for **ICICI** — **Node.js Backend Developer** (~**30 minutes**, **7+ years**).

Simple English. Practical answers. Banking-aware habits (transactions, idempotency, security).

---

## Company / position context

| Item | Detail |
|------|--------|
| Company | ICICI |
| Role | Node.js Backend Developer |
| Round | First technical screen (~30 min) |
| Stack (JD) | Node.js, TypeScript/JS, Express, Sequelize, SQL, async, API security, Jest, Git, Postman |
| What they probe | Correct APIs under concurrency — not trivia, not mega distributed systems |

Expect depth on **correctness under concurrency**, **SQL/Sequelize**, **async behavior**, and **safe money-like operations**, plus a short talk about **your real experience**.

---

## How to use this guide (2 days)

1. Follow **`00-two-day-plan.md`** — Critical blocks first.
2. Study topic files **01–11** in plan order. Fill **`14-experience.md`** with **your** facts only.
3. Drill **`12-scenarios.md`** out loud.
4. Skim **`13-system-design.md`** at first-round depth.
5. Run **`16-mock-interview.md`** twice under a timer.
6. Last hour: only **`15-revision-sheet.md`**.

---

## Priority legend

| Tag | Meaning |
|-----|---------|
| `[Critical]` | Likely asked; a weak answer risks the round |
| `[High]` | Common; strong answers separate seniors |
| `[Medium]` | Nice coverage; don’t steal time from Critical |

---

## Full file map

| File | Focus |
|------|--------|
| [00-two-day-plan.md](./00-two-day-plan.md) | Hour-by-hour 2-day timetable |
| [01-nodejs.md](./01-nodejs.md) | Architecture, Event Loop, streams, cluster, GC + output Qs |
| [02-javascript.md](./02-javascript.md) | Closures, this, promises, array methods, coding pitfalls |
| [03-typescript.md](./03-typescript.md) | Types, generics, narrowing, tsconfig, strict mode |
| [04-express-rest.md](./04-express-rest.md) | Middleware, REST, JWT, pagination, idempotency |
| [05-async-errors.md](./05-async-errors.md) | Errors, races, retries, timeouts, graceful shutdown |
| [06-sequelize.md](./06-sequelize.md) | **JD-critical** — models, associations, txs, N+1 (+ TypeORM notes) |
| [07-sql.md](./07-sql.md) | Indexes, joins, ACID, isolation, EXPLAIN, locking |
| [08-security.md](./08-security.md) | JWT, injection, CORS, bcrypt, Helmet, PII |
| [09-performance.md](./09-performance.md) | Redis, scaling, pools, queues, memory |
| [10-testing-git.md](./10-testing-git.md) | Jest, Postman, merge/rebase, reset/revert |
| [11-coding.md](./11-coding.md) | JS + SQL coding drills with solutions |
| [12-scenarios.md](./12-scenarios.md) | Production / API judgment scenarios |
| [13-system-design.md](./13-system-design.md) | First-round design building blocks |
| [14-experience.md](./14-experience.md) | STAR templates (fill with your facts) |
| [15-revision-sheet.md](./15-revision-sheet.md) | Last-hour one-line cheat sheet |
| [16-mock-interview.md](./16-mock-interview.md) | Timed 30-min mock + scoring |

---

## Top Critical topics for this JD

1. **Node.js event loop** — non-blocking I/O; what blocks the process  
2. **Express** — middleware order, async errors, clean API structure  
3. **Sequelize + SQL** — transactions, associations, N+1, indexes, `EXPLAIN`  
4. **Async control** — `Promise.all`, timeouts, retries, concurrency limits  
5. **Idempotency** — unique keys; safe retries; no double-charge  
6. **Concurrent updates** — optimistic versioning / row locks  
7. **API security** — authz, injection, secrets, rate limits, PII  
8. **Connection pooling** — `instances × pool.max` < DB max  
9. **Jest** — unit tests with mocks; money-path edge cases  
10. **Production judgment** — slow API triage, crash analysis, graceful failure  

---

## Suggested start path

```
README → 00-two-day-plan → Day 1 Critical (01, 04, 05, 06, 07, 08)
      → 12-scenarios → 13-system-design
      → 14-experience (fill placeholders)
      → 16-mock-interview ×2
      → 15-revision-sheet (last hour)
```

Measure first. Keep writes transactional. Make retries idempotent.
