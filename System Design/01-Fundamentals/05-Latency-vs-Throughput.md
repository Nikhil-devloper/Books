# Latency vs Throughput

## 1. Simple Explanation

**Latency:** how long one request takes (ms). Lower is better for user feel.
**Throughput:** how many requests/operations per second the system handles (RPS/QPS). Higher is better for capacity.

You can have high throughput with bad latency (many slow requests) or low latency with low throughput (fast but few).

## 2. Practical Backend Example

Product details API:
- Latency target: p50 20ms, p99 100ms.
- Throughput: 20k RPS at peak.

Adding heavy logging sync to disk may keep throughput OK early but destroy p99 latency.

## 3. How It Works (step by step)

1. Measure latency as percentiles: p50, p95, p99 (averages hide pain).
2. Measure throughput as RPS / messages/sec.
3. Optimize latency: cache, indexes, avoid chatty calls, connection pooling.
4. Optimize throughput: parallelism, batching, horizontal scale, async offload.

## 5. Why Do We Need It?

Users feel latency. Business capacity planning needs throughput. Interviews expect you to know both.

## 6. Alternatives & Trade-offs

| Goal | Techniques |
|------|------------|
| Lower latency | Cache, CDN, indexes, closer region |
| Higher throughput | More workers, batch writes, partitions |
| Both | Often need architecture changes, not just bigger box |

## 7. Interview Revision Notes

- Latency = time/request. Throughput = requests/time.
- Use p99, not only average.
- Async queues improve API latency by moving slow work off the request path.

## 8. Common Interview Q&A

**Q1. Difference between latency and throughput?**
Latency = delay for one op. Throughput = ops per unit time.

**Q2. Why p99 matters?**
A small % of slow requests still hurt many users at scale and reveal tail issues (GC, locks, cold cache).

## 9. Practical Production Scenario

Search API throughput 5k RPS but p99 = 2s due to missing index. Add index → p99 80ms; same throughput feels "fixed" because latency was the real pain.

## 10. Self-Check Questions

1. Define latency and throughput with units.
2. Why can adding servers raise throughput but not fix p99?
3. How does a queue affect client-facing latency?
