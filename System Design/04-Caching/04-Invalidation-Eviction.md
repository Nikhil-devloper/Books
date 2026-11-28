# Cache Invalidation and Eviction

## 1. Simple Explanation

**Invalidation:** actively removing/updating cache when source data changes.
**Eviction:** cache automatically drops keys (TTL expired or memory full policy).

"Cache invalidation is hard" because it's easy to serve stale or wrong data.

## 2. Practical Backend Example

User updates avatar. DB updated but CDN/Redis still has old URL. Invalidate `user:42` and purge CDN path.

## 3. How It Works (step by step)

Invalidation approaches:
1. Delete keys on write.
2. Short TTL as safety net.
3. Versioned keys (`user:42:v7`) and bump version.
Eviction policies:
- LRU: remove least recently used
- LFU: remove least frequently used
- TTL: time-based expiry

## 5. Why Do We Need It?

Without invalidation users see wrong data; without eviction memory explodes.

## 6. Alternatives & Trade-offs

| Method | Pros | Cons |
|--------|------|------|
| TTL only | Simple | Stale until expiry |
| Delete on write | Fresher | Must know all keys |
| Versioning | Avoids scan deletes | Readers must know version |

## 7. Interview Revision Notes

- TTL + explicit delete is a strong combo.
- Never KEYS * in prod Redis.
- Be precise about stale tolerance.

## 8. Common Interview Q&A

**Q1. Invalidation vs eviction?**
Invalidation = deliberate remove on change. Eviction = policy/TTL memory management.

**Q2. Common eviction policy?**
LRU (and variants).

## 9. Practical Production Scenario

Catalog service cached lists `category:shoes:page:1`. Item price changes; list still stale. Invalidate list keys or stop caching volatile list pages.

## 10. Self-Check Questions

1. Why is invalidation hard?
2. Give one versioning example.
3. What does LRU mean?
