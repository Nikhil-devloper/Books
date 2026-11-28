# 00 — Two-Day Prep Plan (ICICI Node.js First Round)

**Goal:** Be ready for a **30-minute** first technical round (7+ years): Node, Express, Sequelize/SQL, async, security, Jest, and practical scenarios.

**Priority order:** Critical topics first every day. Do not rearrange to start with “fun” deep dives.

**Legend:** `[C]` Critical · `[H]` High · `[M]` Medium

---

## Day 1 — Foundations + SQL + Security

Focus: sound correct on the JD stack. Scenarios start late Day 1.

| Time | Block | What to do | Pri |
|------|-------|------------|-----|
| 09:00–09:15 | Kickoff | Skim `README.md` + this plan. Write 5 bullets: your years, stack, flagship project, one incident, one SQL win. | `[C]` |
| 09:15–10:15 | Node.js core | Event loop, blocking vs non-blocking, streams basics, cluster/process model, errors/`unhandledRejection`. Use revision sheet after each subtopic. | `[C]` |
| 10:15–10:30 | Break | Walk / water. No phone rabbit holes. | — |
| 10:30–11:30 | Express | Middleware order, async error handling, routing structure, validation, central errors, Helmet/CORS. Sketch your preferred folder layout once. | `[C]` |
| 11:30–12:30 | Async mastery | Promises, async/await, `Promise.all` vs sequential, timeouts, retries, backoff, concurrency limits. Write 5 tiny mental examples. | `[C]` |
| 12:30–13:15 | Lunch | Full break. | — |
| 13:15–14:45 | Sequelize + SQL | Models, associations, transactions, N+1/`include`, migrations, pool settings. Run through `EXPLAIN` conceptually. Practice saying a transaction aloud. | `[C]` |
| 14:45–15:00 | Break | | — |
| 15:00–16:00 | Security | AuthN/AuthZ, JWT checks, injection, secrets, rate limits, PII in logs, OWASP highlights for APIs. Tie each point to banking caution. | `[C]` |
| 16:00–16:45 | Jest | Unit vs integration, mocking DB/HTTP, testing failure paths & idempotency. List 8 tests you’d write for a payment service. | `[H]` |
| 16:45–17:00 | Break | | — |
| 17:00–18:00 | Scenarios set 1 | From `12-scenarios.md`: Q1 slow API, Q2 concurrent update, Q3 partial insert, Q4 duplicate payment, Q10 N+1. Answer out loud. | `[C]` |
| 18:00–18:30 | Experience drafts | Fill templates in `14-experience.md` for Q1–Q4 only (intro, project, architecture, prod issue). | `[C]` |
| 18:30–19:00 | Light review | Re-read what you missed today; mark weak spots with a sticky note / list. Stop on time. | `[H]` |

**Day 1 exit criteria:** You can explain event loop, Express errors, a Sequelize transaction, SQL injection defense, and double-charge prevention without notes.

---

## Day 2 — Scenarios + Design + Mock + Polish

Focus: sound practical under time pressure.

| Time | Block | What to do | Pri |
|------|-------|------------|-----|
| 09:00–09:20 | Warm-up | Recite Day 1 Critical list from `15-revision-sheet.md` (Node → Express → Async → SQL → Security). | `[C]` |
| 09:20–10:20 | Scenarios set 2 | `12-scenarios.md`: Q5 external down, Q6 large upload, Q7 100k records, Q8 background email, Q9 DB connection failure. | `[C]` |
| 10:20–10:35 | Break | | — |
| 10:35–11:20 | Scenarios set 3 | Q11 prod crash, Q12 API security, Q13 scaling Node. Re-drill any weak Q from set 1. | `[C]` |
| 11:20–12:20 | System design | Full `13-system-design.md`. Practice the whiteboard sketch in 2 minutes. Drill: LB, H/V scale, Redis + invalidation, indexes/replicas, queues, rate limit, idempotency, pooling, notifications. | `[H]` |
| 12:20–13:05 | Lunch | | — |
| 13:05–13:50 | Experience finish | Complete `14-experience.md` Q5–Q10. Speak each STAR in ≤90 seconds. Record one if possible. | `[C]` |
| 13:50–14:05 | Break | | — |
| 14:05–14:45 | Mock interview #1 | Run `16-mock-interview.md` strictly timed (30 min). Score yourself. | `[C]` |
| 14:45–15:15 | Gap drill | Only items scored 0–1 that are Critical (money safety, transactions, SQL, Node basics). | `[C]` |
| 15:15–15:30 | Break | | — |
| 15:30–16:00 | Mock interview #2 | Second pass with different B/C picks from the mock file. | `[H]` |
| 16:00–16:30 | Jest + security rapid fire | 15 min Jest Q&A aloud; 15 min security checklist from revision sheet. | `[H]` |
| 16:30–17:00 | Buffer / weak topic | Use for your worst area only (often SQL `EXPLAIN` or idempotency races). | `[C]` |
| 17:00–17:30 | Final revision | Read `15-revision-sheet.md` once slowly. Skim scenario table at the bottom. | `[C]` |
| 17:30–18:00 | Logistics & calm | Resume PDF ready, headset/network check if virtual, sleep plan. No new topics. | `[M]` |

**Day 2 exit criteria:** Two mocks done; Critical scenario answers are crisp; experience templates filled with **your** facts; revision sheet feels familiar.

---

## If you only have one day

Do Day 1 Critical blocks (Node, Express, Async, SQL, Security) + Day 2 scenarios set 1–2 + one mock + revision sheet. Skip deep design stretch topics after the sketch.

## If you only have half a day (emergency)

1. `15-revision-sheet.md` (45 min)  
2. Scenarios Q2, Q3, Q4, Q1, Q10, Q12 (60 min)  
3. Fill intro + prod issue + SQL win STAR (30 min)  
4. One 30-min mock (30 min)  
5. Re-read money-safety + transactions (15 min)

---

## Study hygiene

- Prefer **speaking** answers over silent reading.
- Keep examples banking-safe: transactions, idempotency, audit, PII.
- When stuck, say structure first: “I’d measure → isolate → fix → verify.”
- Sleep > cramming past midnight before the interview.
