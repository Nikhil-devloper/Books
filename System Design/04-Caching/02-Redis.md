# Redis

## 1. Simple Explanation

**Redis** is an in-memory data store used as cache, session store, rate limiter, and simple queue/pub-sub. Extremely fast because data lives in RAM.

## 2. Practical Backend Example

NestJS stores session tokens / product JSON in ElastiCache Redis. `GET product:100` returns in sub-ms vs tens of ms from Postgres.

## 3. How It Works (step by step)

1. Client issues GET/SET with TTL.
2. Redis serves from memory.
3. Evicts keys under pressure by policy (LRU etc.).
4. Persistence optional (AOF/RDB) — often disabled/relaxed for pure cache.
5. Cluster mode shards keys across nodes for size/throughput.

## 5. Why Do We Need It?

Need shared fast state across many stateless API instances.

## 6. Alternatives & Trade-offs

| Use | Redis structure |
|-----|-----------------|
| Object cache | STRING/JSON |
| Rate limit | INCR + EXPIRE |
| Leaderboard | ZSET |
| Pub/Sub | PUBLISH/SUBSCRIBE |

## 7. Interview Revision Notes

- Redis = shared memory for distributed apps.
- Always set TTL on cache keys.
- Memory size planning matters.

## 8. Common Interview Q&A

**Q1. Why Redis over local in-process cache?**
Shared across instances; consistent invalidation; survives process restart (if same cluster).

**Q2. Is Redis durable source of truth?**
Usually no for app data — DB is. Redis can be durable but treat carefully.

## 9. Practical Production Scenario

Local Map cache per ECS task shows different prices after update. Move to Redis + delete key on write.

## 10. Self-Check Questions

1. Why is Redis fast?
2. Two non-cache uses of Redis?
3. What happens when Redis is full?
