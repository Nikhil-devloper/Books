# HTTP and HTTPS

## 1. Simple Explanation

**HTTP** is the request/response protocol of the web (methods, status codes, headers, body).
**HTTPS** is HTTP over TLS — encrypted and authenticated channel.

## 2. Practical Backend Example

NestJS `GET /users/42` returns 200 JSON. `POST /orders` returns 201. Client uses HTTPS so tokens/PII are encrypted in transit. ALB terminates TLS and forwards HTTP to tasks in private subnets.

## 3. How It Works (step by step)

1. Client opens TCP connection (and TLS handshake for HTTPS).
2. Sends request line + headers + optional body.
3. Server processes; returns status + headers + body.
4. Connection may keep-alive for reuse.
5. HTTPS adds certificate validation + encryption.

## 5. Why Do We Need It?

Standard app communication. HTTPS protects against eavesdropping and tampering on the network.

## 6. Alternatives & Trade-offs

| | HTTP | HTTPS |
|--|------|-------|
| Encryption | No | Yes (TLS) |
| Use | Local/dev only ideally | Production |

## 7. Interview Revision Notes

- Know methods (GET/POST/PUT/PATCH/DELETE) and idempotency expectations.
- Use HTTPS everywhere in prod.
- Status codes communicate outcomes (2xx/4xx/5xx).

## 8. Common Interview Q&A

**Q1. HTTP vs HTTPS?**
HTTPS = HTTP + TLS encryption/authentication.

**Q2. Why terminate TLS at ALB?**
Central cert management; backend can stay on private HTTP; still encrypt internet path.

## 9. Practical Production Scenario

Mobile app accidentally used HTTP in staging and leaked auth headers on public Wi‑Fi. Prod enforces HTTPS-only and HSTS.

## 10. Self-Check Questions

1. What does TLS protect?
2. Is GET idempotent? Is POST?
3. Meaning of 401 vs 403 vs 429?
