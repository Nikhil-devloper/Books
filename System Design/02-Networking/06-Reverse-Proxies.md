# Reverse Proxies

## 1. Simple Explanation

A **reverse proxy** accepts client requests and forwards them to backend servers. Clients talk to the proxy, not directly to app servers.

Load balancers are often reverse proxies. NGINX/Enovy also do TLS, caching, compression, routing.

## 2. Practical Backend Example

NGINX terminates TLS, serves static assets, proxies `/api` to NestJS on port 3000. App servers stay private.

## 3. How It Works (step by step)

1. Client → proxy.
2. Proxy applies rules (auth headers, rate limit, TLS).
3. Proxy forwards to upstream.
4. Upstream responds; proxy returns to client.

## 5. Why Do We Need It?

Security boundary, TLS centralization, routing, soft buffering of backends.

## 6. Alternatives & Trade-offs

| Tool | Common role |
|------|-------------|
| NGINX | TLS, static, reverse proxy |
| ALB | Managed L7 LB/proxy |
| CloudFront | CDN + reverse proxy at edge |

## 7. Interview Revision Notes

- Reverse proxy ≠ forward proxy (forward is for clients egress).
- Often combined with load balancing.

## 8. Common Interview Q&A

**Q1. What is a reverse proxy?**
Proxy that represents backends to clients.

**Q2. Benefits?**
TLS offload, hiding internals, routing, caching, security filters.

## 9. Practical Production Scenario

Exposing Nest directly to internet on :3000. Move behind NGINX/ALB; only 443 public.

## 10. Self-Check Questions

1. Forward vs reverse proxy?
2. Can a reverse proxy cache responses?
3. Where does TLS terminate?
