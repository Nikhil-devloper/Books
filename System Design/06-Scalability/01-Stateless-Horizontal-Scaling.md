# Stateless Architecture and Horizontal Scaling

## 1. Simple Explanation

**Stateless app server:** does not store user session data in local memory/disk between requests. Any instance can handle any request.
This unlocks easy **horizontal scaling** behind a load balancer.

## 2. Practical Backend Example

Don't store sessions in NestJS memory. Store session/JWT data in Redis or use JWTs. Scale ECS desired count from 2 → 20 during peak.

## 3. How It Works (step by step)

1. Externalize state (DB/Redis/S3).
2. Put ALB in front.
3. Autoscale on CPU/RPS/queue.
4. Rolling deploys without sticky needs.

## 5. Why Do We Need It?

Stateful local sessions break load balancing and autoscaling.

## 6. Alternatives & Trade-offs

| Stateful | Stateless |
|----------|-----------|
| Sticky sessions | Any node works |
| Hard deploys | Easy replace |

## 7. Interview Revision Notes

- Stateless API tier is interview baseline.
- State belongs in shared stores.

## 8. Common Interview Q&A

**Q1. What does stateless mean?**
No required local server memory for request continuity.

**Q2. Where do sessions go?**
Redis/DB or client JWT.

## 9. Practical Production Scenario

Deploy killed in-memory carts. Move carts to Redis keyed by user id.

## 10. Self-Check Questions

1. Why sticky sessions are a smell?
2. What state is OK in memory?
3. Autoscaling metric examples?
