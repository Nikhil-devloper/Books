# Circuit Breakers

## 1. Simple Explanation

A **circuit breaker** stops calling a failing dependency for a while after errors spike — like an electrical breaker — so your app fails fast and recovers later.

## 2. Practical Backend Example

Inventory service is down. Without breaker, Nest checkout threads wait on timeouts and collapse. With breaker open, checkout immediately returns degraded response / queues work.

## 3. How It Works (step by step)

1. Closed: calls flow; track failures.
2. Trip open after threshold.
3. Open: fail fast (no call).
4. Half-open: try few probe calls.
5. Success → close; fail → open again.

## 5. Why Do We Need It?

Prevent cascading failures and resource exhaustion.

## 6. Alternatives & Trade-offs

| Timeout only | Timeout + circuit breaker |
|--------------|---------------------------|
| Still hammers dependency | Gives dependency time to recover |

## 7. Interview Revision Notes

- Pair with timeouts, retries, bulkheads.
- Don't retry blindly while breaker open.

## 8. Common Interview Q&A

**Q1. Circuit breaker states?**
Closed, Open, Half-open.

**Q2. Why fail fast?**
Free resources; improve p99; avoid cascading outage.

## 9. Practical Production Scenario

Payment PSP latency 5s. Breaker opens; site still browses; checkout shows "payments temporarily unavailable."

## 10. Self-Check Questions

1. What trips a breaker?
2. What is half-open?
3. Difference breaker vs retry?
