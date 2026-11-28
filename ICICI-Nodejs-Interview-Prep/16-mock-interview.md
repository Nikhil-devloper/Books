# 16 — 30-Minute Mock Interview

Simulate the **ICICI Node.js Backend Developer — first technical round**. Use a timer. Speak answers out loud. A peer can score you; otherwise self-score honestly.

**Rules:** No notes for Round A–C. Revision sheet allowed only in the 2-minute buffer before start. Stop at 30:00 even mid-answer.

---

## Timed agenda

| Time | Section | Goal |
|------|---------|------|
| 0:00–2:00 | Intro | Clear senior intro |
| 2:00–12:00 | Core Node + Express + SQL | Depth on JD stack |
| 12:00–20:00 | Scenarios | Practical judgment |
| 20:00–26:00 | Light system design | Building blocks |
| 26:00–29:00 | Experience / behavioral | STAR, not stories without results |
| 29:00–30:00 | Your questions | Thoughtful close |

---

## Section A — Intro (2 min)

### A1. Tell me about yourself. `[Critical]`

**Expected points:**
- Years + Node/Express/SQL focus
- Current domain + your ownership
- One concrete achievement
- Why this role (1 line)

**Red flags:** 5-minute life story; no Node mention; reading resume line-by-line.

---

## Section B — Core technical (10 min)

Ask **4 of 6**. Candidate should average ~2 min each.

### B1. What happens when a Node.js HTTP request arrives until the response is sent? `[Critical]`

**Expected points:**
- Request hits Express stack / middleware
- Route handler runs; async I/O yields event loop
- DB/external via drivers/callbacks/promises
- Response `res.json`; connection handling
- Mentions non-blocking vs blocking risk

### B2. Explain promises vs async/await and a concurrency pitfall. `[Critical]`

**Expected points:**
- async/await = syntax over promises
- Sequential awaits vs `Promise.all`
- Error handling with try/catch
- Unhandled rejection danger

### B3. How do you use transactions in Sequelize (or equivalent)? `[Critical]`

**Expected points:**
- `sequelize.transaction` / managed transaction
- Pass `transaction` to all queries
- Rollback on throw
- When needed: multi-table consistency

### B4. How do you prevent SQL injection? `[Critical]`

**Expected points:**
- Bindings / parameterized queries
- Sequelize `where` objects, not string concat
- Input validation still required
- Least-privilege DB user as defense in depth

### B5. How do you structure tests with Jest for an API handler? `[High]`

**Expected points:**
- Unit-test service with mocked repo
- Test success + validation + failure paths
- Mock external HTTP
- Mention integration test value

### B6. JWT auth on an Express route — what do you check? `[High]`

**Expected points:**
- Verify signature + expiry
- Load identity / roles; authorize resource
- HTTPS; short TTL
- Don’t put secrets/PII in token payload carelessly

---

## Section C — Scenarios (8 min)

Ask **3 of 4**. Push one follow-up each.

### C1. Payment API must not double-charge on client retry. Design it. `[Critical]`

**Expected points:**
- Idempotency-Key header
- Unique constraint / store response
- Transaction around ledger write
- Safe interaction with payment gateway status check

**Follow-up:** Two parallel requests with same key? — Unique insert / row lock; one wins, one replays.

### C2. p95 latency became 5s after a release. Your move? `[Critical]`

**Expected points:**
- Check deploy diff + dashboards
- Isolate DB vs external vs CPU
- SQL/`EXPLAIN` / N+1
- Rollback vs hotfix decision
- Verify fix with metrics

### C3. Job must process 100k account rows nightly. `[High]`

**Expected points:**
- No full load into memory
- Batches + keyset pagination
- Queue workers + concurrency limit
- Checkpoint / idempotent retries
- Monitoring of job success

### C4. Upstream KYC API is intermittently timing out. `[High]`

**Expected points:**
- Timeouts set
- Limited retries with backoff (if safe)
- Circuit breaker
- User-safe error / async retry path
- Alerting on dependency SLI

---

## Section D — Light system design (6 min)

### D1. Sketch a Node service that sends transfer success SMS without slowing the API. Add one cache for “frequent payees.” `[High]`

**Expected points (draw + talk):**
- LB → Node API (stateless)
- Write transfer in DB transaction
- Enqueue notification job; worker → SMS provider
- Redis cache for frequent payees; TTL + invalidate on change
- Mentions failure/retry for SMS
- Optional: rate limit on transfer API

**Stretch (only if time):** Read replica for payee list — note lag.

**Red flags:** Diving into Kafka multi-region consensus; ignoring DB consistency on transfer.

---

## Section E — Experience (3 min)

### E1. Hardest production bug you owned? `[Critical]`

**Expected points (STAR):**
- Clear symptom + impact
- Ordered debugging steps
- Root cause (specific)
- Fix + prevention (test/alert/index)

### E2. Why ICICI / why you? (if time) `[Medium]`

**Expected points:**
- Reliable transactional systems interest
- Mapping of skills to JD (Node, SQL, security)
- Senior ownership tone without arrogance

---

## Section F — Candidate questions (1 min)

Good examples:
- How is the service ownership split (payments vs platform)?
- What does on-call look like for this team?
- Node/Sequelize versions and primary datastore?

---

## Scoring notes

Score each asked question **0–2**:

| Score | Meaning |
|------|---------|
| 0 | Wrong, blank, or pure buzzwords |
| 1 | Partially right; missing trade-off or production angle |
| 2 | Correct, practical, concise, with a senior edge |

**Section weights (approximate):**

| Section | Weight |
|---------|--------|
| B Core | 35% |
| C Scenarios | 30% |
| D Design | 15% |
| A+E Experience | 15% |
| Communication clarity | 5% (global modifier) |

**Pass guideline for first round:** ~**70%+** with **no zeros on Critical money/transaction topics** (transactions, idempotency, SQL injection, concurrent update).

**Instant fail patterns:**
- Concatenating SQL with user input
- “Just retry payment POST” without idempotency
- Blocking event loop as a non-issue
- Cannot explain a transaction
- Inventing fake metrics when asked for experience (use templates; say “example structure” if practicing without notes)

---

## Self-run script (solo)

1. Phone timer 30:00.
2. Record voice (optional).
3. Answer A1 → pick B1,B2,B3,B5 → C1,C2,C3 → D1 → E1.
4. Transcribe key gaps into a “missed” list.
5. Re-drill only the 0–1 scored Critical items for 20 minutes.

---

## Quick answer length guide

| Question type | Target length |
|---------------|---------------|
| Definition / concept | 45–60 sec |
| Scenario | 90–120 sec |
| Design sketch | 3–4 min |
| STAR story | 90 sec |
