# Mock System Design Interviews — How to Practice

## Rules

1. Pick one HLD file from `08-HLD-Problems/`.
2. Set a 45–60 minute timer.
3. Speak aloud using the 10-step framework in `00-Interview-Framework.md`.
4. Do **not** open the solution notes until you finish.
5. Score yourself on the rubric below.
6. Re-answer weak sections in 10 minutes.

## Rubric (score 1–5 each)

| Criterion | What good looks like |
|-----------|----------------------|
| Clarifying questions | Scoped FR/NFR before drawing |
| Estimates | Rough QPS/storage with assumptions |
| API & data model | Clear endpoints + keys/indexes |
| Architecture | Coherent components, not buzzword soup |
| Flow narration | End-to-end happy path |
| Bottlenecks | Finds the real hot path |
| Scale plan | Concrete next steps |
| Failure handling | Timeouts, retries, idempotency, SPOFs |
| Trade-offs | Explicit "I chose X because…" |
| Communication | Structured, senior, calm |

Target: average ≥ 4 before interviews.

## Suggested mock order (P5)

1. URL Shortener
2. Rate Limiter
3. Notification System
4. E-commerce
5. WhatsApp Chat
6. Food Delivery
7. Ride Booking
8. Instagram Feed
9. File Storage
10. YouTube

## Partner prompt (give to interviewer friend)

"Act as a strict system design interviewer for a 7+ YOE backend role. Ask one clarifying question at a time if I jump ahead. Challenge trade-offs. Do not teach until the end."

## Self-review checklist after each mock

- Did I quantify anything?
- Did I mention failure modes?
- Did I justify DB choice with access patterns?
- Did I keep the first design simple before scaling?
- Did I talk about idempotency/async where needed?
