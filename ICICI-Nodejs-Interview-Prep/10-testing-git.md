# 10 — Testing & Git (Node.js Backend)

First technical round focus: how you verify APIs and work safely with Git.

---

### Q1. What is Jest and why is it used? `[High]`
**Answer:** Jest is a popular JavaScript test runner for unit and integration tests. It provides assertions, mocks, spies, coverage, and a watch mode out of the box. In Node backends you use it to test services, controllers, and utilities quickly. It works well with Express apps when you also use something like `supertest` for HTTP tests.
**Example:**
```js
// sum.test.js
const sum = (a, b) => a + b;

test('adds two numbers', () => {
  expect(sum(2, 3)).toBe(5);
});
```
**Follow-up:** What is the difference between `toBe` and `toEqual`?

---

### Q2. Unit tests vs integration tests — what is the difference? `[Critical]`
**Answer:** Unit tests check one small piece (function/class) in isolation, usually with dependencies mocked. Integration tests check several parts working together — for example Express route + service + real test database. Unit tests are fast and pinpoint bugs; integration tests catch wiring mistakes. A healthy suite uses many unit tests and fewer, focused integration tests.
**Example:**
```text
Unit:        calculateInterest(principal, rate) with pure inputs
Integration: POST /loans → controller → DB → response shape
```
**Follow-up:** Where do end-to-end (E2E) tests fit compared to these two?

---

### Q3. What is mocking and spying in Jest? `[High]`
**Answer:** Mocking replaces a real dependency with a fake so the test stays focused and fast (for example fake email sender or DB). Spying wraps a real function to record how it was called while optionally keeping original behavior. Use mocks for external I/O; use spies to assert call counts/arguments. Avoid over-mocking or tests only check that mocks call mocks.
**Example:**
```js
const email = require('../email');
jest.mock('../email');

test('sends welcome mail', async () => {
  email.sendWelcome.mockResolvedValue(true);
  await registerUser({ email: 'a@b.com' });
  expect(email.sendWelcome).toHaveBeenCalledWith('a@b.com');
});

test('spy example', () => {
  const spy = jest.spyOn(Math, 'random').mockReturnValue(0.5);
  expect(Math.random()).toBe(0.5);
  spy.mockRestore();
});
```
**Follow-up:** When should you mock the database vs use a test database?

---

### Q4. What is test coverage and how should you use it? `[Medium]`
**Answer:** Coverage shows which lines, branches, and functions tests executed. It is a useful signal, not a goal by itself — 100% coverage can still miss important bugs. Focus coverage on business logic: auth, payments, validations. Use coverage reports to find untested risky paths, not to game a percentage.
**Example:**
```bash
npx jest --coverage
# Look at coverage/lcov-report for uncovered branches
```
**Follow-up:** Why can high coverage still miss a production bug?

---

### Q5. How do you test an Express API with Jest? `[High]`
**Answer:** Use `supertest` to send HTTP requests to your Express `app` without starting a real public server. Assert status codes, headers, and JSON body. Reset database state between tests for integration suites. Keep tests independent so order does not matter.
**Example:**
```js
const request = require('supertest');
const app = require('../app');

describe('GET /health', () => {
  it('returns ok', async () => {
    const res = await request(app).get('/health');
    expect(res.status).toBe(200);
    expect(res.body).toEqual({ status: 'ok' });
  });
});
```
**Follow-up:** Should tests hit production databases? Why or why not?

---

### Q6. What is Postman used for? `[Medium]`
**Answer:** Postman is a tool to manually and automatically call HTTP APIs. Developers use it to explore endpoints, save request collections, set environments (dev/stage tokens), and run basic regression suites. It is great for quick verification and demos; automated CI usually still relies on Jest/supertest or similar. Collections can be shared with the team.
**Example:**
```text
Collection: Payments API
Env: local → baseUrl=http://localhost:3000, token={{authToken}}
Request: POST {{baseUrl}}/payments  Authorization: Bearer {{token}}
```
**Follow-up:** How do Postman environments help when switching between local and staging?

---

### Q7. How do you approach API testing for a banking-style service? `[High]`
**Answer:** Test happy paths and failure paths: auth success/failure, validation errors, authorization (user A cannot see user B), idempotent payments, and rate-limit behavior. Assert status codes and error contract shape, not only “200 OK.” Include negative tests for SQL injection-like inputs and oversized payloads at a basic level. Automate the critical flows in CI.
**Example:**
```text
Cases for POST /transfer:
1) valid transfer → 201
2) missing auth → 401
3) insufficient balance → 400/422
4) transfer to other user's account without rights → 403/404
5) duplicate Idempotency-Key → same result, no double debit
```
**Follow-up:** What is an idempotency key and why do payment APIs need one?

---

### Q8. Git merge vs rebase — what is the difference? `[Critical]`
**Answer:** `git merge` joins two branches and usually creates a merge commit, preserving parallel history. `git rebase` replays your commits on top of another branch, creating a straighter history. Use merge for shared/long-lived branches; rebase is often used to clean local feature commits before merge (with care). Never rebase commits already pushed and used by others unless the team agrees.
**Example:**
```bash
# merge main into feature
git checkout feature
git merge main

# rebase feature onto latest main (local cleanup)
git checkout feature
git fetch origin
git rebase origin/main
```
**Follow-up:** Why is rewriting public history dangerous?

---

### Q9. What is cherry-pick? `[Medium]`
**Answer:** Cherry-pick copies a specific commit from one branch onto your current branch. Useful when you need one bugfix on release without bringing the whole feature branch. It creates a new commit with the same changes (new hash). Resolve conflicts if the surrounding code differs.
**Example:**
```bash
git checkout release/1.2
git cherry-pick abc1234
```
**Follow-up:** What problem can happen if you cherry-pick the same fix into multiple branches?

---

### Q10. Git reset vs revert — when do you use each? `[Critical]`
**Answer:** `reset` moves the branch pointer (and optionally working tree) to another commit — good for fixing local mistakes not yet shared. `revert` creates a new commit that undoes a previous commit — safe for shared/main branches because history stays intact. Prefer revert on main/production branches. Use reset carefully and usually only on local commits.
**Example:**
```bash
# undo last local commit, keep changes staged
git reset --soft HEAD~1

# safely undo a commit on main
git revert <commit-sha>
```
**Follow-up:** What do `--soft`, `--mixed`, and `--hard` mean for reset?

---

### Q11. How do you resolve merge conflicts? `[High]`
**Answer:** A conflict means Git cannot automatically combine changes in the same lines. Open the conflicted files, choose the correct final code (or combine both intents), remove conflict markers, then `git add` and continue the merge/rebase. Talk to the other author if business logic is unclear. After resolving, run tests before pushing.
**Example:**
```text
<<<<<<< HEAD
return res.status(400).json({ error: 'Invalid amount' });
=======
return res.status(422).json({ error: 'Invalid amount' });
>>>>>>> feature/validation
```
```bash
# edit file → choose one version → then:
git add src/payments.js
git merge --continue   # or: git rebase --continue
```
**Follow-up:** How do you abort a merge if you are stuck?

---

### Q12. What is a pull request review checklist for backend code? `[Medium]`
**Answer:** Check correctness, edge cases, authZ, input validation, error handling, and logging of sensitive data. Look for N+1 queries, missing indexes notes, and test coverage for critical paths. Confirm migrations are safe and secrets are not committed. Ask “can this break production data?”
**Example:**
```text
PR checklist:
[ ] Validation on input
[ ] No secrets in diff
[ ] Tests for happy + failure path
[ ] Backward-compatible API/DB change
```
**Follow-up:** What makes a DB migration backward compatible?

---

### Q13. What is the difference between `git fetch` and `git pull`? `[Medium]`
**Answer:** `fetch` downloads remote updates but does not change your working branch. `pull` fetches and then integrates (merge or rebase) into your current branch. Fetch is safer when you want to inspect remote commits first. Pull is convenient but can create unexpected merge commits if you are not careful.
**Example:**
```bash
git fetch origin
git log HEAD..origin/main   # inspect first
git merge origin/main       # integrate when ready
```
**Follow-up:** When would you prefer `git pull --rebase`?

---

### Q14. How do you write a good commit message? `[Medium]`
**Answer:** Keep the subject short and imperative (“Add rate limit to login”). Explain why in the body when the change is non-obvious. One logical change per commit helps review and revert. Avoid vague messages like “fix” or “updates.”
**Example:**
```text
Add Redis caching for GET /accounts/:id

Cuts p95 latency under load by avoiding repeated DB reads.
Invalidates cache on account update.
```
**Follow-up:** Why are small commits easier to revert?

---

### Q15. What is branching strategy at a high level (Git Flow / trunk-based)? `[Medium]`
**Answer:** Teams use short-lived feature branches off `main`/`develop`, then merge via PR. Git Flow uses longer `develop`/`release`/`hotfix` branches; trunk-based development keeps branches very short and ships often. For interviews, stress: protect main, use PRs, CI must pass, prefer small merges. Match whatever the company already uses.
**Example:**
```text
feature/login-rate-limit → PR → main → tag/release
hotfix/payment-double-charge → PR → main (+ cherry-pick to release if needed)
```
**Follow-up:** What is a hotfix branch for?

---

### Q16. How do you keep tests reliable in CI? `[High]`
**Answer:** Make tests deterministic: fixed clocks when needed, isolated data, no dependence on run order or wall-clock flakiness. Use a clean test DB or containers per pipeline. Fail the build on test failure and keep the suite reasonably fast so developers run it often. Quarantine or fix flaky tests quickly.
**Example:**
```js
beforeEach(async () => {
  await db.migrate.latest();
  await db.seed.run();
});

afterEach(async () => {
  await db('users').truncate();
});
```
**Follow-up:** What makes a test “flaky”?

---

### Q17. Stub vs mock vs fake — quick differences? `[Medium]`
**Answer:** A stub returns canned answers. A mock also verifies interactions (was it called?). A fake is a working lightweight substitute (in-memory DB). In practice people say “mock” for all fakes, but being precise shows maturity. Pick the lightest double that keeps the test meaningful.
**Example:**
```text
Stub:  getUser.returns({ id: 1 })
Mock:  expect(getUser).toHaveBeenCalledWith(1)
Fake:  InMemoryUserRepo with real insert/find logic in memory
```
**Follow-up:** Why can heavy mocking make refactors painful?

---

### Q18. How do you test error handling paths? `[High]`
**Answer:** Force failures on purpose: invalid body, expired JWT, DB down (mocked rejection), and permission denied. Assert status code, error code/message shape, and that no partial side effects remain when transactions roll back. Error paths are where production incidents often hide.
**Example:**
```js
test('returns 401 for bad token', async () => {
  const res = await request(app)
    .get('/me')
    .set('Authorization', 'Bearer bad.token');
  expect(res.status).toBe(401);
  expect(res.body.error).toBeDefined();
});
```
**Follow-up:** Should APIs return stack traces to clients in production?

---

## Quick Cheat Sheet

| Topic | Remember |
|-------|----------|
| Unit vs integration | Isolation vs wired-together |
| Mock/spy | Replace deps / observe calls |
| Coverage | Signal, not vanity metric |
| merge vs rebase | Preserve history vs linearize |
| reset vs revert | Local rewrite vs safe undo commit |
| conflicts | Edit markers → add → continue |
| Postman | Manual/exploratory + collections |
| CI tests | Deterministic, isolated, fast enough |
