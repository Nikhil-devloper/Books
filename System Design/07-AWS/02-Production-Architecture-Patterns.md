# Production AWS Architecture Patterns

## 1. Simple Explanation

Common patterns seniors draw in interviews for real backend systems on AWS.

## 2. Practical Backend Example

E-commerce API:
Internet → CloudFront (static) + ALB (API) → ECS services → RDS primary/replicas + Redis → SNS/SQS workers → S3 media → CloudWatch alarms.

## 3. How It Works (step by step)

1. Public subnet: ALB only (and maybe NAT egress).
2. Private subnet: ECS tasks, RDS, Redis.
3. Auth at edge/API (Cognito/JWT).
4. Writes to primary; reads cache-first then replica.
5. Side effects async via SNS/SQS.
6. Metrics/logs/alarms in CloudWatch; page on DLQ depth / 5xx / CPU / free storage.

## 5. Why Do We Need It?

Security, scale, and operability require a standard shape — not random services.

## 6. Alternatives & Trade-offs

| Pattern | Use |
|---------|-----|
| ALB + ECS + RDS | Core business API |
| S3 presigned uploads | Large files without through-app |
| SNS fanout + SQS workers | Event-driven side effects |
| API GW + Lambda | Lightweight serverless APIs |
| Multi-AZ everything critical | Availability |

## 7. Interview Revision Notes

- Private apps, public LB.
- Async side effects.
- Alarms on user-facing symptoms and queue health.

## 8. Common Interview Q&A

**Q1. Why private subnets for ECS/RDS?**
Reduce attack surface; only LB publicly reachable.

**Q2. How to handle secrets?**
Secrets Manager/SSM — not baked into images.

## 9. Practical Production Scenario

5xx spike: CloudWatch alarm → check ALB target health → RDS CPU → Redis evictions → SQS backlog. Patterned observability shortens MTTR.

## 10. Self-Check Questions

1. Draw a secure 3-tier AWS API.
2. Where does TLS terminate?
3. How do workers scale?
