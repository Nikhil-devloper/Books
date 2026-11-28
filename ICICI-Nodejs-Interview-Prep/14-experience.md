# 14 — Experience Answers (STAR Templates)

**Do not invent experience.** Fill the `[brackets]` with your real projects, metrics, and tools. Practice speaking each answer in 60–90 seconds.

STAR reminder: **S**ituation → **T**ask → **A**ction → **R**esult (prefer a measurable result).

---

### Q1. Tell me about yourself. `[Critical]`

**Answer (template):**
I am a backend engineer with **[X]+ years** focused on **Node.js / Express** APIs and **[SQL databases — Postgres/MySQL]**. In my current role at **[Company]**, I build and maintain **[product/domain — e.g. payments, lending, ops platforms]** serving **[users/TPS/scale hint]**. My day-to-day work covers **API design, Sequelize/SQL, async processing, and production reliability**. Recently I **[1 concrete win: reduced latency / led migration / stabilized incidents]**. I am interested in ICICI’s Node.js role because **[banking-scale systems / reliable transactional backends / impact]** and my background maps well to **Node, Express, SQL, and secure API delivery**.

**Follow-up:** Be ready to deepen any one project they pick from this intro.

---

### Q2. Tell me about your current / recent project. `[Critical]`

**Answer (STAR template):**
- **Situation:** At **[Company]**, we built/maintained **[system name]** which does **[one-sentence purpose]** for **[customers/internal users]**.
- **Task:** My ownership was **[APIs / module / service]** — specifically **[create transfer API / partner integration / reporting service]**.
- **Action:** I designed **[endpoints + data model]**, used **[Express + Sequelize/raw SQL]**, handled **[auth, validation, transactions]**, and integrated **[queue/Redis/third-party]**. Collaborated with **[FE / QA / DevOps]** on **[contracts, releases]**.
- **Result:** **[metric: latency p95, error rate, throughput, delivery date, incident reduction]** and what I learned about **[reliability / SQL / async]**.

**Follow-up:** Draw a simple box diagram of services you touched.

---

### Q3. Explain the backend architecture of a system you worked on. `[Critical]`

**Answer (template):**
- **Clients:** **[web/mobile/internal]** call our APIs through **[API gateway / LB]**.
- **App layer:** **[N]** Node.js services — mine owned **[service names]**. Stateless behind **[LB/k8s]**.
- **Data:** Primary DB **[Postgres/MySQL]** with **[Sequelize/Knex/raw]**; caching in **[Redis]** for **[what]**; files in **[S3/…]**.
- **Async:** **[Bull/SQS/Kafka]** for **[emails, statements, webhooks]**.
- **Cross-cutting:** Auth via **[JWT/OAuth/session]**, logging/metrics via **[tool]**, deploy via **[CI/CD + env]**.
- **Why this shape:** **[simple reason: separation of reads/writes, blast-radius, scale]**.

**Follow-up:** If asked “what would you change?” — give one honest improvement (`[better idempotency / clearer bounded contexts / stronger observability]`).

---

### Q4. Describe a difficult production issue you handled. `[Critical]`

**Answer (STAR template):**
- **Situation:** In **[prod/env]**, we saw **[symptom: timeouts / 5xx spike / wrong balances / memory growth]** starting **[when]**.
- **Task:** I needed to **restore service** and **find root cause** without making it worse.
- **Action:**
  1. Checked **[dashboards/logs/alerts]** — narrowed to **[API / DB / dependency]**.
  2. Found **[root cause: missing index / pool exhaustion / bad deploy / N+1 / poison message]**.
  3. Mitigated with **[rollback / feature flag / scale-out / kill switch / hotfix]**.
  4. Permanent fix: **[migration/index/code change/tests/alert]**.
- **Result:** Recovered in **[time]**; **[metric improved]**; added **[guardrail]** so it does not repeat.

**Follow-up:** Emphasize calm triage order: impact → mitigate → root cause → prevent.

---

### Q5. How do you debug issues in Node.js backends? `[High]`

**Answer (template):**
My default loop:
1. **Reproduce** with request id / correlation id from logs.
2. Check **symptoms**: error rate, latency, dependency failures, recent deploys.
3. Narrow layer: **gateway → app → DB → external**.
4. Use **structured logs**, SQL logging/`EXPLAIN` for DB, and APM traces if available.
5. For local hard bugs: debugger / targeted tests / feature flags.
6. For memory/CPU: heap snapshots, event-loop lag metrics.
I document findings and add a **regression test or alert** when the bug is subtle.

**Fill-in:** Tools I actually use — `[CloudWatch / Datadog / New Relic / ELK / Grafana / clinic.js]`.

**Follow-up:** Give one real example in 30 seconds using the production-issue STAR.

---

### Q6. Tell me about a DB / query optimization you did. `[Critical]`

**Answer (STAR template):**
- **Situation:** Endpoint/report **[X]** was slow (**[e.g. 3–5s]** ) under **[load/data size]**.
- **Task:** Bring p95 under **[target]**.
- **Action:** Measured with **[slow query log / EXPLAIN / APM]**. Found **[full table scan / bad JOIN / N+1 / SELECT *]**. Fixed by **[index on (a,b) / rewrite query / batching / pagination / covering index]**. Validated in staging with production-like data.
- **Result:** Latency **[before → after]**; write impact checked; index documented in migration **[name]**.

**Follow-up:** Be ready to explain the `EXPLAIN` idea: seq scan vs index scan, rows estimate.

---

### Q7. How do you design APIs? Give an example. `[High]`

**Answer (template):**
I start from **use cases and resources**, not random URLs.
- Clear resources: `POST /payments`, `GET /payments/:id`.
- Contract: request/response schema, error shape, status codes (`201`, `400`, `401`, `409`, `429`).
- Validation at the edge; authz before business logic.
- For money APIs: **idempotency key**, strict audit fields, idempotent retries.
- Versioning strategy we used: **[`/v1` URL or header]** because **[reason]**.
Example from my work: **[endpoint]** that did **[behavior]**, with **[pagination / filtering / transaction boundary]** decisions.

**Follow-up:** How you handle breaking changes — `[version bump / expand-contract]`.

---

### Q8. How do you approach code reviews? `[Medium]`

**Answer (template):**
I review for **correctness first** (logic, race conditions, transaction boundaries), then **security** (authz, injection, secrets), then **performance** (N+1, unbounded queries), then **readability**. I prefer small PRs with tests for edge cases. As an author I add context in the PR description and call out risky areas. As a reviewer I ask questions, suggest alternatives, and avoid nitpicking style when a linter exists. Example: I once caught **[missing transaction / missing index / leaking PII in logs]** before prod.

**Follow-up:** What do you do when you disagree? — Discuss with data; spike if needed; escalate only for risk.

---

### Q9. Explain your deployment / release process experience. `[High]`

**Answer (template):**
- **CI:** On PR — lint, unit tests (**Jest**), **[build]**.
- **CD:** Merge to main → build image → deploy to **[dev/staging]** → smoke tests → prod with **[rolling / blue-green]**.
- Config via **[env / secret manager]**; DB changes via **[migrations — Sequelize/Flyway]** run **[before/during]** release with expand-contract when needed.
- Rollback: **[redeploy previous image / feature flag off]**.
- My role: **[owned pipeline piece / wrote migration / monitored release]**.
For banking-like caution I emphasize **migration safety**, **feature flags**, and **post-deploy checks**.

**Follow-up:** Worst deploy lesson — fill with one real story briefly.

---

### Q10. Why should we hire you for this ICICI Node.js role? `[High]`

**Answer (template):**
You need someone who can ship **secure, reliable Node/Express APIs** on **SQL** with strong instincts for **transactions, idempotency, and production debugging**. That matches my **[X] years** building **[similar systems]**. I am comfortable with **Sequelize/SQL, async/queues, Jest, and security basics**, and I communicate trade-offs clearly. I learn domain rules quickly — important in banking — and I care about **operability** (logs, metrics, safe releases), not only features. I want to contribute to **[ICICI / digital banking backends]** where correctness matters as much as speed.

**Follow-up:** Prepare 1–2 questions for them about team stack, on-call, and domain (payments/cards/lending).

---

## Filling checklist (do this before the interview)

| Slot | Your real detail |
|------|------------------|
| Years + stack | `[ ]` |
| Flagship project + your module | `[ ]` |
| One architecture diagram (boxes only) | `[ ]` |
| One prod incident STAR with metric | `[ ]` |
| One SQL optimization STAR with before/after | `[ ]` |
| One security-conscious API example | `[ ]` |
| Tools: APM, CI, queue, Redis, cloud | `[ ]` |
| Why ICICI (1 honest sentence) | `[ ]` |
