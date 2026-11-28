# At-least-once vs At-most-once vs Exactly-once

## 1. Simple Explanation

**At-most-once:** may lose messages; never duplicates.
**At-least-once:** may duplicate; shouldn't lose (common).
**Exactly-once:** processed once effectively — hard; usually "exactly-once effect" via idempotency + dedupe.

## 2. Practical Backend Example

SQS standard: at-least-once. Your worker might see the same order event twice → must be idempotent.
Logging metrics where losing one datapoint is OK → at-most-once acceptable.

## 3. How It Works (step by step)

1. Producer sends.
2. Consumer crashes after side effect but before ack → message redelivered (at-least-once).
3. If ack before side effect and crash → loss (at-most-once behavior).
4. Exactly-once needs transactional outbox / idempotent sink / dedupe store.

## 5. Why Do We Need It?

Distributed systems fail; delivery semantics decide correctness design.

## 6. Alternatives & Trade-offs

| Guarantee | Loss | Dupes | Typical |
|-----------|------|-------|---------|
| At-most-once | Possible | No | Metrics |
| At-least-once | Rare/no | Possible | SQS, most queues |
| Exactly-once effect | No | No effective | Idempotent + dedupe |

## 7. Interview Revision Notes

- Interview golden line: "We assume at-least-once and make consumers idempotent."
- True end-to-end exactly-once is rare/expensive.

## 8. Common Interview Q&A

**Q1. Which does SQS provide?**
At-least-once (standard); FIFO has stronger ordering/dedupe within limits.

**Q2. How approximate exactly-once?**
Idempotency keys + unique constraints + transactional outbox.

## 9. Practical Production Scenario

Worker sends email then crashes before delete message → duplicate email. Fix: store `email_sent_for_order_id` uniquely before send, or use provider idempotency.

## 10. Self-Check Questions

1. Define at-least-once.
2. Why exactly-once is hard?
3. What app pattern handles duplicates?
