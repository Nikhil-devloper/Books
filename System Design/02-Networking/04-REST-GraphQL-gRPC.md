# REST vs GraphQL vs gRPC

## 1. Simple Explanation

**REST:** resource-based HTTP APIs (URLs + methods). Simple, ubiquitous.
**GraphQL:** client asks for exact fields in one query endpoint.
**gRPC:** high-performance RPC using HTTP/2 + protobuf; great service-to-service.

## 2. Practical Backend Example

Public mobile API: REST `/products/1`.
Complex frontend needing many shapes: GraphQL.
Internal NestJS → NestJS order/inventory calls at high QPS: gRPC.

## 3. How It Works (step by step)

REST: map entities to routes; return JSON; version via URL/headers.
GraphQL: schema + resolvers; client query defines payload.
gRPC: define `.proto` contracts; generate clients; streaming possible.

## 5. Why Do We Need It?

Different clients and network chatty-ness need different API styles.

## 6. Alternatives & Trade-offs

| | REST | GraphQL | gRPC |
|--|------|---------|------|
| Ease | High | Medium | Medium |
| Over/under fetch | Common | Controlled | Efficient binary |
| Browser support | Native | Via HTTP | Limited / grpc-web |
| Best for | Public APIs | Flexible clients | Internal microservices |

## 7. Interview Revision Notes

- Default public API: REST.
- GraphQL when many clients need custom shapes.
- gRPC for internal low-latency service calls.
- Don't force GraphQL/gRPC without a pain REST can't solve.

## 8. Common Interview Q&A

**Q1. When choose gRPC over REST?**
Internal services needing low latency, strong contracts, streaming.

**Q2. GraphQL downside?**
Complexity, caching difficulty, noisy neighbor queries, N+1 resolver risks.

## 9. Practical Production Scenario

BFF for mobile was 6 REST round-trips. GraphQL cut to 1 call. Inventory↔Order internal path moved to gRPC for p99.

## 10. Self-Check Questions

1. One strength of each.
2. Why is REST still default?
3. Caching: easier in REST or GraphQL?
