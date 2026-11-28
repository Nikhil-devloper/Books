# Functional vs Non-Functional Requirements

## 1. Simple Explanation

**Functional requirements (FR):** what the system must do — features and behaviors.
**Non-functional requirements (NFR):** how well it must do it — speed, uptime, scale, security, cost.

Example FR: "User can place an order."
Example NFR: "Place order API p99 < 300ms; 99.9% availability."

## 2. Practical Backend Example

URL shortener:
- FR: create short URL; redirect; optional analytics.
- NFR: redirect < 100ms; highly available; read-heavy (e.g. 10:1 or 100:1 reads/writes).

Your NestJS service may implement FR correctly and still fail interviews if you ignore NFRs (latency/availability under load).

## 3. How It Works (step by step)

1. Ask: what features are in / out of scope?
2. Ask: expected users, QPS, latency targets, consistency needs?
3. Write FR as user actions.
4. Write NFR as measurable targets (ms, %, TB, QPS).
5. Let NFRs drive architecture (cache, replicas, multi-AZ).

## 5. Why Do We Need It?

FR without NFR → pretty design that collapses under traffic.
NFR without FR → over-engineering the wrong product.

## 6. Alternatives & Trade-offs

| Type | Focus | Example |
|------|-------|---------|
| FR | Features | Login, upload, search |
| NFR | Quality attributes | Latency, availability, durability |
| Constraints | Hard limits | Must use AWS; budget; compliance |

## 7. Interview Revision Notes

- FR = what. NFR = how well.
- Always quantify NFRs when possible.
- Consistency is an NFR: strong vs eventual.

## 8. Common Interview Q&A

**Q1. Difference between FR and NFR?**
FR = features. NFR = performance/reliability/scale/security qualities.

**Q2. Example NFRs for chat?**
Message delivery latency, online presence freshness, 99.9%+ availability, ordering per conversation.

**Q3. Why clarify NFRs early?**
They decide SQL vs NoSQL, cache, multi-region, sync vs async.

## 9. Practical Production Scenario

Interviewer: Design Instagram feed. Weak candidate lists features only. Strong candidate asks: freshness SLA? celebrity fan-out? read QPS? storage for media?

## 10. Self-Check Questions

1. Give 3 FRs and 3 NFRs for e-commerce checkout.
2. Is "use PostgreSQL" an FR or a constraint/decision?
3. How does "redirect must be < 100ms" change design?
