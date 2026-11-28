# 08 — Security (Node.js Backend)

First technical round focus: practical security concepts for APIs, auth, and data handling.

---

### Q1. How do you keep JWTs secure in a Node.js API? `[Critical]`
**Answer:** Use short-lived access tokens (for example 15 minutes) and longer-lived refresh tokens stored securely. Sign JWTs with a strong secret or asymmetric keys (RS256), never put passwords or PII in the payload, and always verify signature and expiry on every request. Prefer `httpOnly` + `Secure` cookies for browser clients so JavaScript cannot read the token. Rotate secrets and invalidate refresh tokens on logout or password change.
**Example:**
```js
const jwt = require('jsonwebtoken');

const accessToken = jwt.sign(
  { sub: user.id, role: user.role },
  process.env.JWT_ACCESS_SECRET,
  { expiresIn: '15m' }
);

function authMiddleware(req, res, next) {
  const token = req.cookies.accessToken || req.headers.authorization?.split(' ')[1];
  try {
    req.user = jwt.verify(token, process.env.JWT_ACCESS_SECRET);
    next();
  } catch {
    return res.status(401).json({ error: 'Unauthorized' });
  }
}
```
**Follow-up:** What is the difference between storing JWT in localStorage vs httpOnly cookie?

---

### Q2. What is SQL injection and how do you prevent it? `[Critical]`
**Answer:** SQL injection happens when user input is concatenated into a SQL string and the database runs attacker-controlled queries. Always use parameterized queries or prepared statements; never build SQL with string concatenation. ORMs like Sequelize/Prisma use bind parameters by default, which is safer. Also validate and sanitize input types before they reach the database layer.
**Example:**
```js
// BAD — injectable
const rows = await db.query(`SELECT * FROM users WHERE email = '${email}'`);

// GOOD — parameterized
const rows = await db.query('SELECT * FROM users WHERE email = $1', [email]);
```
**Follow-up:** Can ORM usage alone make you safe from all injection issues?

---

### Q3. What is XSS and how do you protect a Node.js backend? `[High]`
**Answer:** Cross-Site Scripting (XSS) is when an attacker injects malicious JavaScript into pages viewed by other users. On the backend, escape or encode user-generated content before storing or returning it, and set a strong Content-Security-Policy. Prefer frameworks that auto-escape templates (like EJS with escaping enabled). Never trust client-rendered HTML from user input without sanitization.
**Example:**
```js
const escapeHtml = (str) =>
  String(str)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;');

app.post('/comments', (req, res) => {
  const safe = escapeHtml(req.body.text);
  // store and return safe text
});
```
**Follow-up:** What is the difference between stored XSS and reflected XSS?

---

### Q4. What is CSRF and how do you prevent it? `[High]`
**Answer:** Cross-Site Request Forgery tricks a logged-in user’s browser into sending unwanted requests to your API. It mainly affects cookie-based auth because the browser sends cookies automatically. Mitigate with CSRF tokens (synchronizer token or double-submit cookie), `SameSite=Strict/Lax` cookies, and checking the `Origin`/`Referer` header. For pure Bearer-token APIs in headers, classic CSRF risk is lower because the browser does not auto-attach those tokens.
**Example:**
```js
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });

app.get('/form', csrfProtection, (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

app.post('/transfer', csrfProtection, (req, res) => {
  // request only succeeds with valid CSRF token
});
```
**Follow-up:** Why does SameSite cookie attribute help against CSRF?

---

### Q5. What is CORS and how should you configure it? `[High]`
**Answer:** CORS (Cross-Origin Resource Sharing) tells browsers which other origins may call your API. Never use `Access-Control-Allow-Origin: *` with credentials. Allow only trusted domains, specific methods, and needed headers. Handle preflight (`OPTIONS`) correctly and keep the allowlist short in production.
**Example:**
```js
const cors = require('cors');

app.use(cors({
  origin: ['https://app.bank.example'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true,
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
```
**Follow-up:** What happens if CORS is misconfigured but a non-browser client (Postman/curl) still works?

---

### Q6. How does rate limiting protect an API? `[High]`
**Answer:** Rate limiting caps how many requests a client can make in a time window, which reduces brute-force login attempts, scraping, and denial-of-service load. Common keys are IP address, user ID, or API key. Use Redis-backed counters for multi-instance apps so limits are shared across servers. Return `429 Too Many Requests` with a clear retry hint.
**Example:**
```js
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: { error: 'Too many login attempts. Try later.' },
});

app.post('/login', loginLimiter, loginHandler);
```
**Follow-up:** How would you rate-limit differently for authenticated vs anonymous traffic?

---

### Q7. How does bcrypt work for password hashing? `[Critical]`
**Answer:** bcrypt is a slow, adaptive hashing algorithm designed for passwords. It automatically salts each hash so identical passwords produce different hashes. The cost (work factor) controls how slow hashing is; higher cost means harder brute force. Always hash on the server and compare with `bcrypt.compare`, never with string equality on raw hashes in a naive way that skips constant-time comparison (bcrypt’s compare handles this).
**Example:**
```js
const bcrypt = require('bcrypt');

const hash = await bcrypt.hash(password, 12);
const ok = await bcrypt.compare(password, hash);
```
**Follow-up:** Why is a high bcrypt cost both good and potentially risky for API latency?

---

### Q8. Password hashing vs encryption — what is the difference? `[Critical]`
**Answer:** Hashing is one-way: you cannot get the original password back; you only verify by hashing again and comparing. Encryption is two-way: with the key you can decrypt and recover the original value. Passwords must be hashed (plus salted), not encrypted, so a database leak does not expose usable passwords. Use encryption for data you later need to read back (for example some card metadata under strict controls), not for login passwords.
**Example:**
```text
Password flow:  plain → bcrypt.hash → store hash → bcrypt.compare on login
Secret config:  plain → encrypt(key) → store ciphertext → decrypt(key) when needed
```
**Follow-up:** If someone asks to “decrypt the password,” what does that tell you about the design?

---

### Q9. What does Helmet do in Express? `[Medium]`
**Answer:** Helmet sets secure HTTP response headers that reduce common browser attacks. Examples include `X-Content-Type-Options`, `X-Frame-Options` / frameguard, and Content-Security-Policy helpers. It is a quick baseline hardening step, not a full security solution. You still need auth, validation, and safe coding practices.
**Example:**
```js
const helmet = require('helmet');
app.use(helmet());
```
**Follow-up:** Name two headers Helmet helps set and what each prevents.

---

### Q10. How do you validate input in a Node.js API? `[Critical]`
**Answer:** Validate every external input: body, query, params, and headers. Reject unexpected fields and enforce types, lengths, and formats early (fail fast). Libraries like Joi, Zod, or express-validator keep rules clear and reusable. Never rely on frontend validation alone.
**Example:**
```js
const { z } = require('zod');

const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(128),
  age: z.number().int().min(18).max(120),
});

app.post('/users', (req, res) => {
  const parsed = createUserSchema.safeParse(req.body);
  if (!parsed.success) {
    return res.status(400).json({ errors: parsed.error.flatten() });
  }
  // use parsed.data only
});
```
**Follow-up:** What is the difference between validation and sanitization?

---

### Q11. What are rules for secure logging? `[High]`
**Answer:** Never log passwords, tokens, OTPs, full card numbers, or Aadhaar/PAN-like identifiers. Mask or hash identifiers when logs need correlation. Keep log levels sensible in production and send logs to a secured store with access control. Avoid logging entire request bodies by default.
**Example:**
```js
function safeUserLog(user) {
  return {
    id: user.id,
    email: user.email.replace(/(^.).*(@.*$)/, '$1***$2'),
    // never: password, token, cvv
  };
}

logger.info('login_success', safeUserLog(user));
```
**Follow-up:** How would you debug a production auth issue without logging secrets?

---

### Q12. How should sensitive data be handled in a banking-style backend? `[Critical]`
**Answer:** Minimize what you store; if you do not need it, do not keep it. Encrypt sensitive fields at rest, use TLS in transit, and restrict access with least privilege (app DB users, IAM roles). Mask data in APIs and UI (show only last 4 digits). Separate secrets from code using environment variables or a secret manager — never commit `.env` with real keys.
**Example:**
```js
function maskAccount(accountNumber) {
  return accountNumber.replace(/\d(?=\d{4})/g, '*'); // ******1234
}

// TLS only, secrets from env/secret manager
const dbUrl = process.env.DATABASE_URL;
```
**Follow-up:** What is the difference between data at rest and data in transit encryption?

---

### Q13. How do you secure API keys and secrets in Node.js? `[High]`
**Answer:** Store secrets in environment variables or a vault (AWS Secrets Manager, HashiCorp Vault), not in source control. Load them at startup and fail fast if required secrets are missing. Rotate keys regularly and use different secrets per environment (dev/stage/prod). Limit who can read production secrets.
**Example:**
```js
function requireEnv(name) {
  const value = process.env[name];
  if (!value) throw new Error(`Missing required env: ${name}`);
  return value;
}

const JWT_SECRET = requireEnv('JWT_ACCESS_SECRET');
```
**Follow-up:** Why is committing a `.env` file with production secrets dangerous even in a private repo?

---

### Q14. What is the principle of least privilege for backend services? `[Medium]`
**Answer:** Each service, DB user, and API token should have only the permissions it needs — nothing more. A reporting service should not get write access to payment tables. If a component is compromised, limited permissions reduce blast radius. Review roles during design reviews and after incidents.
**Example:**
```text
App DB user: SELECT/INSERT/UPDATE on app schema only (no DROP, no other DBs)
Admin migration user: used only in CI/CD migration jobs, not by the running API
```
**Follow-up:** How would you apply least privilege between microservices?

---

### Q15. How do you protect against brute-force login attacks? `[High]`
**Answer:** Combine rate limiting, account lockout or exponential backoff after failed attempts, and strong password hashing (bcrypt/argon2). Add MFA for sensitive accounts when possible. Monitor failed-login spikes and alert on anomalies. Never reveal whether an email exists in error messages (“Invalid credentials” only).
**Example:**
```js
if (await redis.incr(`fail:${email}`) === 1) {
  await redis.expire(`fail:${email}`, 900);
}
if (Number(await redis.get(`fail:${email}`)) > 5) {
  return res.status(429).json({ error: 'Account temporarily locked' });
}
```
**Follow-up:** Why is “email not found” vs “wrong password” a security smell?

---

### Q16. What HTTP security headers matter for APIs serving browsers? `[Medium]`
**Answer:** Use `Strict-Transport-Security` (HSTS) to force HTTPS, `X-Content-Type-Options: nosniff` to block MIME sniffing, and a tight CSP when returning HTML. `Cache-Control: no-store` helps for responses with personal data. Helmet sets many of these defaults for Express apps.
**Example:**
```js
res.set({
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
  'X-Content-Type-Options': 'nosniff',
  'Cache-Control': 'no-store',
});
```
**Follow-up:** Should JSON APIs still care about CSP?

---

### Q17. How do you safely handle file uploads? `[Medium]`
**Answer:** Validate file type by content (magic bytes), not only by extension or `Content-Type`. Limit file size, store uploads outside the web root or in object storage, and generate random stored names. Scan for malware when required and never execute uploaded files on the server.
**Example:**
```js
const multer = require('multer');
const upload = multer({
  limits: { fileSize: 2 * 1024 * 1024 }, // 2MB
  fileFilter(_req, file, cb) {
    if (!['image/jpeg', 'image/png'].includes(file.mimetype)) {
      return cb(new Error('Only JPEG/PNG allowed'));
    }
    cb(null, true);
  },
});
```
**Follow-up:** Why is trusting `file.mimetype` alone not enough?

---

### Q18. What is IDOR and how do you prevent it? `[High]`
**Answer:** Insecure Direct Object Reference means a user can access another user’s resource by changing an ID in the URL. Always authorize: check that the authenticated user owns or may access the resource. Do not rely on “hidden” or hard-to-guess IDs alone. Centralize authZ checks in service/middleware layers.
**Example:**
```js
app.get('/accounts/:id', auth, async (req, res) => {
  const account = await Account.findById(req.params.id);
  if (!account || account.userId !== req.user.id) {
    return res.status(404).json({ error: 'Not found' });
  }
  res.json(account);
});
```
**Follow-up:** Why return 404 instead of 403 for some authorization failures?

---

## Quick Cheat Sheet

| Topic | Do | Don't |
|-------|----|-------|
| JWT | Short TTL, verify always | Store secrets in payload |
| SQL | Parameterized queries | String-concat SQL |
| Passwords | bcrypt/argon2 hash | Encrypt or store plain |
| CORS | Explicit allowlist | `*` with credentials |
| Logs | Mask PII | Log tokens/passwords |
| Input | Validate + sanitize | Trust client only |
