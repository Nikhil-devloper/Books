# Cache Strategies: Cache-Aside, Write-Through, Write-Behind

## 1. Simple Explanation

**Cache-aside (lazy):** app reads cache; on miss reads DB and fills cache. App writes DB then deletes/updates cache.
**Write-through:** app writes to cache and DB together (cache library/DB layer).
**Write-behind (write-back):** write cache first; async flush to DB later (faster writes, more risk).

## 2. Practical Backend Example

Most NestJS APIs use cache-aside with Redis:
```
get(id):
  cached = redis.get(id)
  if cached: return cached
  row = db.find(id)
  redis.setex(id, 60, row)
  return row
```

## 3. How It Works (step by step)

Cache-aside steps:
1. Read cache.
2. Miss → read DB.
3. Populate cache with TTL.
4. On update → write DB → invalidate key.

Write-through: update cache + DB in the write path.
Write-behind: acknowledge after cache write; queue DB write.

## 5. Why Do We Need It?

Different read/write patterns need different consistency/performance balances.

## 6. Alternatives & Trade-offs

| Strategy | Pros | Cons |
|----------|------|------|
| Cache-aside | Simple, common | Extra app logic; stampede risk |
| Write-through | Cache always warm on write | Slower writes |
| Write-behind | Fast writes | Durability risk if cache dies |

## 7. Interview Revision Notes

- Default interview answer: cache-aside + TTL + invalidate on write.
- Mention stampede protection (singleflight/lock/jitter).

## 8. Common Interview Q&A

**Q1. Explain cache-aside.**
App manages cache: miss loads DB and fills cache; writes invalidate.

**Q2. When write-behind?**
Very high write rate where rare data loss is acceptable (carefully) — uncommon for core financial data.

## 9. Practical Production Scenario

After deploy, thundering herd on empty cache. Add jittered TTLs + request coalescing for hot keys.

## 10. Self-Check Questions

1. Difference cache-aside vs write-through?
2. What is cache stampede?
3. Why delete key on write instead of update sometimes?
