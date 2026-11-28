# Dead-Letter Queues, Retries, Idempotency

## 1. Simple Explanation

**Retry:** process again after failure (with backoff).
**DLQ (dead-letter queue):** place for messages that failed too many times.
**Idempotency:** processing the same message/request N times has the same effect as once.

## 2. Practical Backend Example

Payment webhook hits twice. Without idempotency, user charged twice. Store `event_id` uniquely; second time returns previous result.
SQS: after 5 receives failing, message goes to DLQ for investigation.

## 3. How It Works (step by step)

1. Consumer fails.
2. Message becomes visible again after visibility timeout.
3. Retry with exponential backoff / limited attempts.
4. Exceed max → DLQ.
5. Idempotency key / unique constraint prevents duplicate side effects.

## 5. Why Do We Need It?

Networks duplicate; at-least-once delivery is common; poison messages exist.

## 6. Alternatives & Trade-offs

| Without idempotency | With |
|---------------------|------|
| Duplicate emails/charges | Safe retries |
| Fear of retries | Retries become safe |

## 7. Interview Revision Notes

- Assume duplicates will happen.
- Idempotency keys for payments and consumers.
- DLQ + alarms for poison messages.

## 8. Common Interview Q&A

**Q1. What is idempotency?**
Safe to retry; result doesn't multiply side effects.

**Q2. What is a DLQ?**
Queue for messages that repeatedly fail processing.

## 9. Practical Production Scenario

Inventory service throws on bad SKU payload forever. Without DLQ, queue blocks/hot loops. DLQ isolates poison message; alarm notifies.

## 10. Self-Check Questions

1. Why visibility timeout matters in SQS?
2. How to implement idempotency in Postgres?
3. What do you do with DLQ messages?
