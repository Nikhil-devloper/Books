# CDN (Content Delivery Network)

## 1. Simple Explanation

A **CDN** caches static (and sometimes dynamic) content at edge locations close to users so downloads are faster and origin load is lower.

## 2. Practical Backend Example

Product images on S3 behind CloudFront. User in India gets image from nearby edge instead of always hitting ap-south-1 origin.

## 3. How It Works (step by step)

1. User requests asset via CDN domain.
2. Edge checks cache.
3. If miss, fetch from origin (S3/ALB), cache it, return.
4. TTL / invalidation controls freshness.

## 4. Architecture Diagram

```mermaid
flowchart LR
  User --> Edge[CDN Edge]
  Edge -->|miss| Origin[S3/ALB]
  Edge -->|hit| User
```

## 5. Why Do We Need It?

Latency and origin offload for media/static JS/CSS.

## 6. Alternatives & Trade-offs

| Without CDN | With CDN |
|-------------|----------|
| All hits origin | Most hits edge |
| Higher latency globally | Lower TTFB for static |

## 7. Interview Revision Notes

- CDN for static/media first.
- Cache-Control headers matter.
- Invalidate carefully on deploys.

## 8. Common Interview Q&A

**Q1. What problem does CDN solve?**
Bring content closer; reduce origin load/latency.

**Q2. CDN for REST JSON APIs?**
Sometimes for public GETs with careful cache headers; not for personalized private data by default.

## 9. Practical Production Scenario

Launch spike melts origin bandwidth for images. CloudFront absorbs repeat downloads.

## 10. Self-Check Questions

1. What is an edge location?
2. What is cache invalidation?
3. S3 + CloudFront typical flow?
