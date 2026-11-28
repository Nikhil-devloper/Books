# Rate Limiting

## 1. Simple Explanation

**Rate limiting** caps how many requests a client can make in a time window to protect the system and ensure fair use.

## 2. Practical Backend Example

Login endpoint: 5 attempts / 15 min / IP. API key: 1000 req/min. Implemented with Redis counters or token bucket; return 429.

## 3. How It Works (step by step)

1. Identify key (IP, userId, apiKey).
2. On request, increment counter / take token.
3. If over limit → 429 Retry-After.
4. Else proceed.
5. Store must be shared (Redis) across instances.

## 5. Why Do We Need It?

Stops abuse, accidental loops, and noisy neighbors from melting APIs.

## 6. Alternatives & Trade-offs

| Algorithm | Idea |
|-----------|------|
| Fixed window | Count per interval |
| Sliding window | Smoother |
| Token bucket | Burst friendly |
| Leaky bucket | Steady outflow |

## 7. Interview Revision Notes

- Use Redis for distributed limits.
- Different limits for auth vs public read.
- Edge + app layer defense in depth.

## 8. Common Interview Q&A

**Q1. How to rate limit across many Nest instances?**
Shared Redis counter/bucket — not in-memory Map.

**Q2. HTTP status?**
429 Too Many Requests.

## 9. Practical Production Scenario

Competitor scrapes product API at 200 RPS. Rate limit + WAF rules protect origin.

## 10. Self-Check Questions

1. Fixed vs sliding window?
2. Why not memory map?
3. What header help clients?
