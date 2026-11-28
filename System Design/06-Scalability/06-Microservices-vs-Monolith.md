# Microservices vs Monolith

## 1. Simple Explanation

**Monolith:** one deployable unit (modular code still possible).
**Microservices:** many small services, each owning data/logic, deployed independently.

## 2. Practical Backend Example

Early shop: NestJS modular monolith + Postgres — fastest delivery.
Later: split Video Transcoding and Search because they scale/fail differently.

## 3. How It Works (step by step)

Monolith path:
1. One codebase/DB.
2. Simple transactions/calls.
3. Scale by running more copies (if stateless).

Microservices path:
1. Split by domain.
2. Own DBs; communicate via API/events.
3. Independent scale; distributed complexity.

## 5. Why Do We Need It?

Different team size, scale, and failure domains need different shapes.

## 6. Alternatives & Trade-offs

| Monolith | Microservices |
|----------|---------------|
| Simple ops | Independent deploys |
| Easy ACID | Distributed data hard |
| Scale as a whole | Scale parts |
| Risk: big ball of mud | Risk: distributed mess |

## 7. Interview Revision Notes

- Start monolith unless a clear reason to split.
- Split for independent scale/failure/team ownership.
- Microservices without strong boundaries make things worse.

## 8. Common Interview Q&A

**Q1. When microservices?**
Clear domains, independent scaling, multiple teams, proven bottlenecks.

**Q2. Biggest microservice cost?**
Distributed transactions, observability, ops complexity, latency.

## 9. Practical Production Scenario

Team split into 20 services too early; latency and outages rose. Consolidated into 3 domain services + async events.

## 10. Self-Check Questions

1. Modular monolith meaning?
2. Does microservices automatically mean scale?
3. How do services share data safely?
