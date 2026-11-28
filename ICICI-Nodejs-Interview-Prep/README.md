# ICICI Node.js Backend Developer — Interview Prep

First **technical round** prep pack for **ICICI Bank** — **Node.js Backend Developer** (about **30 minutes**, **7+ years** experience level).

Simple English. Practical answers. Banking-aware habits (transactions, idempotency, security).

---

## Company / position context

| Item | Detail |
|------|--------|
| Company | ICICI Bank |
| Role | Node.js Backend Developer (IC-level / experienced) |
| Round | First technical screen (~30 min) |
| Stack emphasis (JD) | Node.js, Express, Sequelize, SQL, async patterns, API security, Jest |
| What they probe | Can you build and operate correct APIs — not trivia, not mega distributed systems |

Expect depth on **correctness under concurrency**, **SQL/Sequelize**, **async behavior**, and **safe handling of money-like operations**, plus a short talk about **your real experience**.

---

## How to use this guide

1. Start with **`00-two-day-plan.md`** and follow Critical blocks first.
2. Study topic files in priority order (see file map). Fill **`14-experience.md`** with **your** facts only — do not invent.
3. Drill **`12-scenarios.md`** out loud; keep answers action-oriented.
4. Skim **`13-system-design.md`** at first-round depth (LB, cache, queue, pool).
5. Run **`16-mock-interview.md`** twice under a real timer.
6. Last hour: only **`15-revision-sheet.md`**.

---

## Priority legend

| Tag | Meaning |
|-----|---------|
| `[Critical]` | Likely asked; a weak answer risks the round |
| `[High]` | Common; strong answers separate seniors |
| `[Medium]` | Nice coverage; don’t steal time from Critical |

---

## File map

| File | Purpose |
|------|---------|
| [00-two-day-plan.md](./00-two-day-plan.md) | Hour-by-hour 2-day timetable + breaks |
| [12-scenarios.md](./12-scenarios.md) | Practical production / API scenarios |
| [13-system-design.md](./13-system-design.md) | First-round system design (not advanced DS) |
| [14-experience.md](./14-experience.md) | STAR templates / placeholders for your career |
| [15-revision-sheet.md](./15-revision-sheet.md) | Dense last-hour cheat sheet |
| [16-mock-interview.md](./16-mock-interview.md) | Timed 30-min mock + scoring |

> If your folder also has earlier numbered topic notes (01–11), use them for deeper study; this pack (00 + 12–16) is enough to drive the final 2-day push and the mock.

---

## Top Critical topics for the ICICI JD

1. **Node.js event loop** — non-blocking I/O; what blocks the process  
2. **Express** — middleware, async errors, clean API structure  
3. **Sequelize + SQL** — transactions, associations, N+1, indexes, `EXPLAIN`  
4. **Async control** — `Promise.all`, timeouts, retries, concurrency limits  
5. **Idempotency & double-charge prevention** — unique keys, safe retries  
6. **Concurrent updates** — optimistic versioning / row locks  
7. **API security** — authz, injection, secrets, rate limits, PII  
8. **Connection pooling** — size vs instance count; fail fast  
9. **Jest** — unit tests with mocks; edge cases for money paths  
10. **Production judgment** — slow API triage, crash analysis, graceful dependency failure  

---

## Suggested order if starting today

```
README → 00-two-day-plan → (Day 1 Critical study)
      → 12-scenarios → 13-system-design
      → 14-experience (fill placeholders)
      → 16-mock-interview ×2
      → 15-revision-sheet (last hour)
```

Good luck — measure, keep writes transactional, and make retries idempotent.
