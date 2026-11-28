# Express & REST APIs — ICICI Node.js Backend Interview Prep

Expect practical Express architecture questions: middleware order, error handling, REST design, auth, and production API concerns (versioning, pagination, CORS, idempotency). Use short answers with keywords and be ready to sketch code.

---

### Q1. Describe Express architecture for a production API. `[Critical]`
**Answer:** Express is a minimal HTTP framework built around an app, a middleware pipeline, and routers. A typical layout separates routes, controllers (or handlers), services (business logic), and data access. Cross-cutting concerns — logging, auth, validation, CORS, rate limiting — live as middleware. Keep handlers thin; put domain rules in services.
**Example:**
```text
src/
  app.ts          # createExpressApp(), mount middleware + routers
  server.ts       # listen(), graceful shutdown
  routes/         # path → controller wiring
  controllers/    # parse req, call service, send res
  services/       # business logic
  middlewares/    # auth, validate, errorHandler
  models/ or repos/
```
```ts
import express from "express";
import { accountRouter } from "./routes/accounts";
import { errorHandler } from "./middlewares/errorHandler";

const app = express();
app.use(express.json());
app.use("/api/v1/accounts", accountRouter);
app.use(errorHandler);
```
**Follow-up:** Why separate `app` and `server`? → Easier unit/integration tests without binding a port; shutdown hooks stay in `server`.

---

### Q2. What is middleware in Express? `[Critical]`
**Answer:** Middleware is a function `(req, res, next) => {}` that runs in sequence. It can read/modify the request or response, end the cycle by sending a response, or call `next()` / `next(err)` to continue. Types include application-level, router-level, built-in (`express.json`), third-party (`cors`, `helmet`), and error-handling middleware with four arguments.
**Example:**
```ts
function requestId(req, res, next) {
  req.id = crypto.randomUUID();
  res.setHeader("X-Request-Id", req.id);
  next();
}

app.use(requestId);
```
**Follow-up:** What if you forget `next()`? → Request hangs until timeout; client never gets a response.

---

### Q3. How does middleware execution order work? `[Critical]`
**Answer:** Express runs middleware in registration order for matching paths. The first handler that sends a response ends the happy path. Error middleware runs only when `next(err)` is called or a synchronous throw occurs in the stack. Place body parsers and CORS early; auth before protected routes; 404 after all routes; error handler last.
**Example:**
```ts
app.use(cors());
app.use(express.json());
app.use(requestLogger);
app.use("/api/v1/public", publicRouter);
app.use("/api/v1", authMiddleware, protectedRouter);
app.use(notFoundHandler);   // 404
app.use(errorHandler);      // must be last
```
**Follow-up:** Does route-specific middleware run before app-level? → No — matching app-level middleware registered earlier still runs first; order of `app.use` / `router.use` is what matters.

---

### Q4. How do routes and controllers fit together? `[High]`
**Answer:** Routers map HTTP method + path to controller functions. Controllers extract params/query/body, call services, and map results to status codes and JSON. Controllers should not embed SQL or heavy business rules. Use `Router({ mergeParams: true })` when nested routes need parent params.
**Example:**
```ts
// routes/accounts.ts
import { Router } from "express";
import * as AccountController from "../controllers/accounts";
import { requireAuth } from "../middlewares/auth";
import { validate } from "../middlewares/validate";
import { createAccountSchema } from "../schemas/account";

export const accountRouter = Router();
accountRouter.get("/", requireAuth, AccountController.list);
accountRouter.post("/", requireAuth, validate(createAccountSchema), AccountController.create);
accountRouter.get("/:id", requireAuth, AccountController.getById);

// controllers/accounts.ts
export async function getById(req, res, next) {
  try {
    const account = await accountService.findById(req.params.id);
    if (!account) return res.status(404).json({ error: "Not found" });
    res.json(account);
  } catch (err) {
    next(err);
  }
}
```
**Follow-up:** Sync vs async controllers? → Always forward async errors via `try/catch` + `next(err)` or a wrapper; Express 4 does not catch rejected promises automatically (Express 5 does).

---

### Q5. How do you write error-handling middleware? `[Critical]`
**Answer:** Error middleware has signature `(err, req, res, next)`. It must be registered after routes. Map known domain errors to HTTP status codes; hide internal details in production; log with request id. Call `next(err)` from earlier middleware/handlers to reach it.
**Example:**
```ts
function errorHandler(err, req, res, next) {
  const status = err.statusCode || 500;
  const payload = {
    error: status === 500 ? "Internal Server Error" : err.message,
    requestId: req.id,
  };
  if (process.env.NODE_ENV !== "production" && status === 500) {
    payload.stack = err.stack;
  }
  console.error({ err, requestId: req.id });
  res.status(status).json(payload);
}

app.use(errorHandler);
```
**Follow-up:** Multiple error handlers? → Possible; call `next(err)` to pass along. Usually one centralized mapper is clearer.

---

### Q6. Walk through the Express request lifecycle. `[High]`
**Answer:** Client hits the server → Node HTTP creates IncomingMessage/ServerResponse → Express matches the middleware/router chain → body parsers populate `req.body` → route handler runs → service/DB work → response is sent (`res.json` / `res.send`) → remaining middleware is skipped unless errors occur. If no route matches, a 404 handler should run; uncaught errors go to error middleware.
**Example:**
```text
Request → CORS → JSON parser → Logger → Auth → Validate → Controller → Service → DB
                                                                              ↓
Response ← JSON serializer ← Controller ← Service ←──────────────────────────┘
                ↑
         Error handler (if next(err))
```
**Follow-up:** Can middleware run after `res.json`? → Not for writing the body; use `res.on("finish")` for post-response logging/metrics.

---

### Q7. What are the core REST principles? `[High]`
**Answer:** REST is resource-oriented: nouns in URLs (`/accounts/123`), verbs in HTTP methods, stateless requests (each call carries auth/context), uniform interface, and representations (usually JSON). Use hypermedia only if your org adopts HATEOAS; most banking APIs are pragmatic REST — consistent resources, status codes, and error shapes.
**Example:**
```text
GET    /api/v1/accounts          # list
POST   /api/v1/accounts          # create
GET    /api/v1/accounts/:id      # read
PATCH  /api/v1/accounts/:id      # partial update
DELETE /api/v1/accounts/:id      # delete (or soft-close)
POST   /api/v1/accounts/:id/freeze  # non-CRUD action as sub-resource
```
**Follow-up:** Statelessness and JWT? → Server does not store session per request; token carries claims; revocation needs denylist/short TTL/refresh rotation.

---

### Q8. Which HTTP methods matter for APIs? `[Critical]`
**Answer:** `GET` (read, safe, idempotent), `POST` (create/actions, not idempotent by default), `PUT` (full replace, idempotent), `PATCH` (partial update), `DELETE` (remove, idempotent in effect), `HEAD`/`OPTIONS` for metadata and CORS preflight. Do not use `GET` for state-changing operations.
**Example:**
```ts
router.get("/accounts/:id", getAccount);
router.post("/accounts", createAccount);
router.put("/accounts/:id", replaceAccount);
router.patch("/accounts/:id", patchAccount);
router.delete("/accounts/:id", closeAccount);
```
**Follow-up:** Is DELETE always idempotent? → Semantically yes: repeating DELETE leaves the resource absent; second call may return 404 or 204 depending on API policy.

| Method | Safe | Idempotent | Typical use |
|---|---|---|---|
| GET | Yes | Yes | Read |
| POST | No | No* | Create / actions |
| PUT | No | Yes | Full replace |
| PATCH | No | Usually** | Partial update |
| DELETE | No | Yes | Remove |

\*Can be made idempotent with Idempotency-Key.  
\*\*Depends on patch semantics.

---

### Q9. Common HTTP status codes you should know? `[Critical]`
**Answer:** Use precise codes so clients and gateways can react correctly. 2xx success, 4xx client fault, 5xx server fault. Prefer `201` + `Location` on create, `204` for no body, `400` validation, `401` unauthenticated, `403` forbidden, `404` missing, `409` conflict, `422` semantic validation (optional), `429` rate limit, `503` unavailable.
**Example:**
```ts
res.status(201).location(`/api/v1/accounts/${id}`).json(account);
res.status(204).send();
res.status(409).json({ error: "Account already exists" });
```
**Follow-up:** 401 vs 403? → 401 = missing/invalid credentials; 403 = identity known but not allowed.

| Code | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized (unauthenticated) |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Unprocessable Entity |
| 429 | Too Many Requests |
| 500 | Internal Server Error |
| 502/503/504 | Bad gateway / Unavailable / Timeout |

---

### Q10. PUT vs PATCH? `[High]`
**Answer:** `PUT` replaces the entire resource representation; omitted fields are typically cleared or rejected as incomplete. `PATCH` applies a partial update to specified fields only. For banking profiles, `PATCH` is usually safer to avoid wiping unset fields. Both should validate and authorize carefully.
**Example:**
```ts
// PUT — full replace
{ "name": "Asha", "email": "asha@bank.com", "phone": "999" }

// PATCH — only email changes
{ "email": "asha.new@bank.com" }
```
**Follow-up:** Idempotency? → PUT is naturally idempotent; PATCH should be designed to be idempotent where possible (same patch twice → same state).

| | PUT | PATCH |
|---|---|---|
| Body | Full resource | Partial fields |
| Missing fields | Treated as absent/cleared | Left unchanged |
| Idempotent | Yes | Design-dependent |
| Use when | Client has complete snapshot | Client updates few fields |

---

### Q11. GET vs POST — when is POST correct for reads? `[Medium]`
**Answer:** Prefer `GET` for retrieval so responses can be cached and bookmarked; query strings carry filters. Use `POST` for reads only when the query is huge, sensitive (avoid secrets in URLs/logs), or the operation is really an action (search jobs, complex reports). Never use `GET` to mutate balances or trigger transfers.
**Example:**
```ts
// Good
GET /api/v1/transactions?accountId=1&from=2024-01-01

// Acceptable for complex search body
POST /api/v1/transactions/search
{ "accountIds": ["1","2"], "filters": { ... } }
```
**Follow-up:** Caching? → GET responses may be cached by browsers/CDNs/proxies; POST usually is not — important for financial data freshness controls.

---

### Q12. How do you version APIs? `[High]`
**Answer:** Common strategies: URI versioning (`/api/v1/...`), header versioning (`Accept: application/vnd.bank.v2+json`), or query (`?version=2`). URI versioning is the most visible and easiest for gateways. Keep old versions stable; deprecate with headers/docs; avoid breaking changes inside a major version.
**Example:**
```ts
app.use("/api/v1/accounts", accountsV1);
app.use("/api/v2/accounts", accountsV2);
```
**Follow-up:** When to bump major version? → Breaking response shape, removed fields, or changed auth/error contracts — not for additive optional fields.

| Strategy | Pros | Cons |
|---|---|---|
| URI `/v1` | Clear, cache-friendly | URL churn |
| Header | Clean URLs | Harder to test/debug |
| Query `?v=` | Simple | Easy to misuse/cache poorly |

---

### Q13. How do you paginate list APIs? `[High]`
**Answer:** Pagination limits payload size and DB load. Two main styles: offset/limit (page-based) and cursor/keyset (opaque token pointing to last seen row). Always return total or `nextCursor` metadata and enforce a max page size. Sort must be stable for predictable pages.
**Example:**
```ts
// Offset
GET /accounts?limit=20&offset=40

// Cursor
GET /accounts?limit=20&cursor=eyJpZCI6IjEyMyJ9

res.json({
  data: rows,
  page: { limit, offset, total },
  // or: page: { nextCursor, hasMore }
});
```
**Follow-up:** Default limits? → Always set server-side defaults and caps (e.g. max 100) so clients cannot request millions of rows.

---

### Q14. Offset vs cursor pagination? `[High]`
**Answer:** Offset is simple (`LIMIT/OFFSET`) but slow on large offsets and unstable when rows insert/delete mid-scroll. Cursor (keyset) uses a unique sort key (`WHERE id > :lastId ORDER BY id LIMIT n`) — consistent and efficient for infinite scroll and large tables. Prefer cursor for transaction ledgers.
**Example:**
```sql
-- Offset (page 500 can be expensive)
SELECT * FROM tx ORDER BY created_at DESC LIMIT 20 OFFSET 10000;

-- Cursor
SELECT * FROM tx
WHERE (created_at, id) < ($lastCreatedAt, $lastId)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```
**Follow-up:** Can users jump to page 50 with cursors? → Not easily; offset is better for “go to page N”, cursor for “next/prev”.

| | Offset | Cursor |
|---|---|---|
| Implementation | Easy | Medium |
| Deep pages | Slow / inconsistent | Fast / stable |
| Jump to page N | Natural | Hard |
| Real-time inserts | Duplicates/skips | Stable with unique key |
| Best for | Admin UIs, small data | Feeds, ledgers, mobile |

---

### Q15. What is CORS and how do you configure it in Express? `[High]`
**Answer:** CORS (Cross-Origin Resource Sharing) is a browser security mechanism controlling which origins may call your API. Servers send headers like `Access-Control-Allow-Origin`. Preflight `OPTIONS` checks methods/headers. In Express, use the `cors` package; never reflect arbitrary origins with credentials in production.
**Example:**
```ts
import cors from "cors";

app.use(cors({
  origin: ["https://bank.example.com"],
  methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
  allowedHeaders: ["Content-Type", "Authorization", "Idempotency-Key"],
  credentials: true,
  maxAge: 600,
}));
```
**Follow-up:** Do mobile native apps need CORS? → No — CORS is enforced by browsers; server auth is still required.

---

### Q16. How should request validation work? `[Critical]`
**Answer:** Validate early in middleware before business logic. Check body, params, and query against a schema (Zod, Joi, AJV, celebrate). Reject with `400`/`422` and a structured error list. Never trust client input for types, ranges, or HTML/SQL content — validation is not a substitute for parameterized queries and authz.
**Example:**
```ts
import { z } from "zod";

const createTransferSchema = z.object({
  fromAccountId: z.string().uuid(),
  toAccountId: z.string().uuid(),
  amount: z.number().positive().finite(),
  currency: z.enum(["INR", "USD"]),
});

function validate(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({ errors: result.error.issues });
    }
    req.body = result.data; // typed, stripped unknown keys if using .strict()
    next();
  };
}

router.post("/transfers", validate(createTransferSchema), createTransfer);
```
**Follow-up:** Validate at gateway and service? → Yes for defense in depth; at least once at API edge and again for critical money-moving services.

---

### Q17. Authentication vs authorization? `[Critical]`
**Answer:** Authentication answers “who are you?” (login, JWT, mTLS, API keys). Authorization answers “what are you allowed to do?” (RBAC, ABAC, scopes, resource ownership). Always authenticate first, then authorize per route/resource. A valid token does not imply permission to debit any account.
**Example:**
```ts
app.post("/transfers", authenticate, authorize("transfers:create"), createTransfer);

async function authorizeOwnership(req, res, next) {
  const account = await accounts.findById(req.params.id);
  if (!account || account.ownerId !== req.user.id) {
    return res.status(403).json({ error: "Forbidden" });
  }
  next();
}
```
**Follow-up:** RBAC vs ABAC? → RBAC = roles/permissions; ABAC = attributes (branch, amount limit, time) — banks often combine both.

| | Authentication | Authorization |
|---|---|---|
| Question | Who are you? | What can you do? |
| Failure code | 401 | 403 |
| Examples | Password, OTP, JWT | Roles, scopes, ACLs |
| Order | First | After identity known |

---

### Q18. How does JWT authentication work in Express? `[Critical]`
**Answer:** After login, the server issues a signed JWT (header.payload.signature) containing claims (`sub`, `exp`, `roles`). Client sends `Authorization: Bearer <token>`. Middleware verifies signature and expiry with the secret/public key, then attaches `req.user`. Prefer short-lived access tokens; never store sensitive PII in claims; use HTTPS only.
**Example:**
```ts
import jwt from "jsonwebtoken";

function authenticate(req, res, next) {
  const header = req.headers.authorization;
  if (!header?.startsWith("Bearer ")) {
    return res.status(401).json({ error: "Missing token" });
  }
  try {
    req.user = jwt.verify(header.slice(7), process.env.JWT_ACCESS_SECRET);
    next();
  } catch {
    return res.status(401).json({ error: "Invalid token" });
  }
}

// issue
const accessToken = jwt.sign(
  { sub: user.id, roles: user.roles },
  process.env.JWT_ACCESS_SECRET,
  { expiresIn: "15m" }
);
```
**Follow-up:** JWT logout? → Access tokens are stateless until expiry; use short TTL + refresh revocation store/denylist for logout and compromise response.

---

### Q19. Access token vs refresh token? `[High]`
**Answer:** Access tokens are short-lived and sent on every API call. Refresh tokens are longer-lived, stored more carefully (httpOnly secure cookie or secure storage), and used only against a token endpoint to mint new access tokens. Rotate refresh tokens on use; revoke on logout/theft. Keep scopes minimal on access tokens.
**Example:**
```ts
app.post("/auth/refresh", async (req, res) => {
  const refresh = req.cookies.refreshToken;
  const stored = await refreshStore.get(refresh);
  if (!stored || stored.revoked) return res.status(401).end();

  await refreshStore.revoke(refresh); // rotation
  const newRefresh = await refreshStore.issue(stored.userId);
  const accessToken = signAccess(stored.userId);
  res.cookie("refreshToken", newRefresh, { httpOnly: true, secure: true, sameSite: "strict" });
  res.json({ accessToken });
});
```
**Follow-up:** Where to store tokens in SPAs? → Access in memory; refresh in httpOnly cookie — avoid localStorage for refresh tokens (XSS risk).

| | Access token | Refresh token |
|---|---|---|
| Lifetime | Minutes | Hours–days |
| Sent to | All APIs | Auth/token endpoint only |
| Storage | Memory / short-lived | httpOnly cookie / secure store |
| Leak impact | Limited window | High — must revoke/rotate |

---

### Q20. What is idempotency and why do banks need it? `[Critical]`
**Answer:** An idempotent operation produces the same side effect even if retried. Networks timeout; clients retry; without idempotency keys, duplicate POSTs can double-charge. Clients send `Idempotency-Key`; server stores key → response/result for a TTL and replays the first outcome on retries.
**Example:**
```ts
async function idempotency(req, res, next) {
  if (req.method !== "POST") return next();
  const key = req.headers["idempotency-key"];
  if (!key) return res.status(400).json({ error: "Idempotency-Key required" });

  const cached = await redis.get(`idem:${req.user.id}:${key}`);
  if (cached) {
    const { status, body } = JSON.parse(cached);
    return res.status(status).json(body);
  }

  const originalJson = res.json.bind(res);
  res.json = (body) => {
    redis.set(
      `idem:${req.user.id}:${key}`,
      JSON.stringify({ status: res.statusCode, body }),
      "EX",
      24 * 3600
    );
    return originalJson(body);
  };
  next();
}

router.post("/transfers", authenticate, idempotency, createTransfer);
```
**Follow-up:** Scope of the key? → Per user/client + key; same key with different body should return `409` conflict, not silently reuse.
