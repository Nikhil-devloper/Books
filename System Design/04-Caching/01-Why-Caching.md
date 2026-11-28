# Why Caching Is Required

## 1. Simple Explanation

A **cache** stores frequently used data in a fast place (usually memory) so you don't hit the slow source (DB/API) every time.

## 2. Practical Backend Example

Product detail page hit 10k RPS. Postgres primary cannot serve that many identical reads cheaply. Redis caches `product:{id}` for 60s → DB load collapses.

## 3. How It Works (step by step)

1. Identify hot read data.
2. Store it in fast storage with a key.
3. On read: check cache → if miss, load DB → fill cache.
4. Expire or invalidate on changes.

## 4. Architecture Diagram

```mermaid
flowchart LR
  API --> Redis{(Cache)}
  Redis -->|miss| DB[(Postgres)]
  DB --> Redis
```

## 5. Why Do We Need It?

Databases are correct but slower/costlier per read. Caches buy latency and throughput.

## 6. Alternatives & Trade-offs

| Without cache | With cache |
|---------------|------------|
| Every read → DB | Many reads → memory |
| Higher latency/cost | Possible stale data |

## 7. Interview Revision Notes

- Cache hot, read-heavy, slightly stale-OK data.
- Always plan invalidation.
- Cache is not source of truth.

## 8. Common Interview Q&A

**Q1. Why cache?**
Reduce latency and backend load for repeated reads.

**Q2. What should not be cached carelessly?**
Highly sensitive secrets; data that must always be exact without a freshness strategy (e.g. live balance) unless designed carefully.

## 9. Practical Production Scenario

Flash sale: same product ID requested endlessly. Cache + short TTL keeps checkout API alive.

## 10. Self-Check Questions

1. What is a cache hit/miss?
2. Why can cache increase bugs?
3. Name 3 cacheable items in e-commerce.
