# Eventual Consistency

## 1. Simple Explanation

**Eventual consistency** means replicas/systems may temporarily disagree after a write, but if writes stop, they eventually converge to the same value.

## 2. Practical Backend Example

DynamoDB eventually consistent read may not see a write made 10ms ago.
SQS-triggered email status updates inventory service later — UI may briefly show "email pending."

## 3. How It Works (step by step)

1. Write accepted at one place.
2. Propagate asynchronously to others.
3. Readers may see old/new values during window.
4. Conflict resolution if concurrent writes (LWW, vectors, app merge).
5. Converges when propagation completes.

## 5. Why Do We Need It?

Enables higher availability and partition tolerance / lower latency at global scale.

## 6. Alternatives & Trade-offs

| Strong | Eventual |
|--------|----------|
| Simpler mental model | Stale reads possible |
| More coordination | Better availability/latency |

## 7. Interview Revision Notes

- Always state the inconsistency window implications.
- Use for non-critical data paths.
- Combine with idempotent consumers.

## 8. Common Interview Q&A

**Q1. What is eventual consistency?**
System converges over time; interim stale reads allowed.

**Q2. Interview example where it's OK?**
Like counts, recommendation caches, email delivery status.

## 9. Practical Production Scenario

Follower count increments via async events. User refreshes and sees old count for 1–2s — acceptable product-wise.

## 10. Self-Check Questions

1. When is eventual consistency unacceptable?
2. What is a conflict?
3. How do clients cope with stale reads?
