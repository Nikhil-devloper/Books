# Reliability and Fault Tolerance

## 1. Simple Explanation

**Reliability:** system correctly does what it should over time.
**Fault tolerance:** system continues operating when some parts fail.

Failures are normal: disks die, networks blip, deploys go bad, dependencies timeout.

## 2. Practical Backend Example

Payment API depends on a third-party PSP. If PSP times out and your NestJS waits 30s per request, your workers pile up and your API dies too.

Fault-tolerant design: short timeouts, retries with backoff for safe ops, circuit breaker, queue for async confirmation updates, clear user-facing degraded mode.

## 3. How It Works (step by step)

1. Assume dependencies fail.
2. Set timeouts on every external call.
3. Retry only idempotent operations (or use idempotency keys).
4. Use circuit breakers to stop calling a sick dependency.
5. Replicate critical data.
6. Isolate failures (bulkheads): don't let email outage kill checkout.

## 5. Why Do We Need It?

One bad dependency should not cascade into a full outage.

## 6. Alternatives & Trade-offs

| Pattern | Purpose |
|---------|---------|
| Timeout | Fail fast |
| Retry + backoff | Survive blips |
| Circuit breaker | Protect yourself |
| Bulkhead | Isolate resources |
| Replication | Survive node loss |

## 7. Interview Revision Notes

- Design for failure, not for the happy path only.
- Timeouts + idempotent retries + isolation.
- Reliability ≠ "never fails"; it means controlled failure and recovery.

## 8. Common Interview Q&A

**Q1. How do you make an API fault tolerant?**
Timeouts, retries with idempotency, circuit breakers, health checks, multi-AZ, graceful degradation.

**Q2. What is cascading failure?**
One component slows/fails → callers pile up → more components fail.

## 9. Practical Production Scenario

Image resize worker crashes mid-job. Without fault tolerance jobs are lost. With SQS visibility timeout + retry + DLQ after N fails, work is not lost and poison messages are isolated.

## 10. Self-Check Questions

1. Difference between reliability and fault tolerance?
2. Why are retries dangerous without idempotency?
3. Give one bulkhead example in a Node API.
