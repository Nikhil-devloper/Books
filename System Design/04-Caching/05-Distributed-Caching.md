# Distributed Caching

## 1. Simple Explanation

**Distributed cache** is a cache shared by many app instances (Redis cluster / ElastiCache), so all nodes see the same cached data.

## 2. Practical Backend Example

16 NestJS tasks behind ALB. Local memory cache → inconsistent. ElastiCache Redis Cluster shared by all tasks.

## 3. How It Works (step by step)

1. App instances connect to Redis endpoint.
2. Keys hashed to shards/nodes.
3. Reads/writes go to owning node.
4. Replication provides failover for cache nodes.

## 5. Why Do We Need It?

Horizontal API scale requires shared cache for consistency of cached data and for shared rate limits/sessions.

## 6. Alternatives & Trade-offs

| Local cache | Distributed cache |
|-------------|-------------------|
| Fastest, no network | Shared, consistent across nodes |
| Inconsistent per node | Extra network hop |

## 7. Interview Revision Notes

- Prefer distributed cache for multi-instance apps.
- Local cache OK as L1 with short TTL on top of Redis L2.

## 8. Common Interview Q&A

**Q1. Why distributed cache?**
Shared state across scaled-out app servers.

**Q2. L1/L2 caching?**
Process memory L1 + Redis L2 for ultra-hot keys.

## 9. Practical Production Scenario

Rate limiter in local memory allows N×limit across N tasks. Redis INCR makes global limit correct.

## 10. Self-Check Questions

1. Local vs distributed cache?
2. How are keys placed in Redis Cluster?
3. Can cache replication lag matter?
