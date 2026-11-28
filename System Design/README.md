# System Design Interview Prep — Senior Backend Engineer (7+ YOE)

Notes for backend system design interviews (Node.js / NestJS / PostgreSQL / DynamoDB / AWS).

**Audience:** 7+ years experience. Interviewers expect **decisions, trade-offs, bottlenecks, failure handling** — not tool definitions.

---

## How to use these notes

1. Study **one topic file at a time**.
2. Read Simple Explanation → Example → How It Works → Revision Notes.
3. Practice Interview Q&A out loud.
4. Answer Self-Check questions without looking.
5. Prioritize folders marked **P1–P5** if time is short.

---

## Priority order (if short on time)

| Priority | Topic | Folder |
|----------|-------|--------|
| P1 | Database design, indexing, replication | `03-Databases/` |
| P2 | Caching and Redis | `04-Caching/` |
| P3 | Load balancing and horizontal scaling | `06-Scalability/` + networking LB notes |
| P4 | Queues, Kafka, SQS, async | `05-Messaging/` |
| P5 | HLD: URL shortener, WhatsApp, E-commerce | `08-HLD-Problems/` |

---

## Learning path

### 0. Interview method
- [00-Interview-Framework.md](./00-Interview-Framework.md)
- [REVISION-CHEATSHEET.md](./REVISION-CHEATSHEET.md)

### 1. Fundamentals
- [01-What-is-System-Design.md](./01-Fundamentals/01-What-is-System-Design.md)
- [02-Functional-vs-Non-Functional.md](./01-Fundamentals/02-Functional-vs-Non-Functional.md)
- [03-Scalability-Availability.md](./01-Fundamentals/03-Scalability-Availability.md)
- [04-Reliability-Fault-Tolerance.md](./01-Fundamentals/04-Reliability-Fault-Tolerance.md)
- [05-Latency-vs-Throughput.md](./01-Fundamentals/05-Latency-vs-Throughput.md)
- [06-Horizontal-vs-Vertical-Scaling.md](./01-Fundamentals/06-Horizontal-vs-Vertical-Scaling.md)
- [07-CAP-Theorem.md](./01-Fundamentals/07-CAP-Theorem.md)
- [08-Consistency-Models.md](./01-Fundamentals/08-Consistency-Models.md)
- [09-Single-Point-of-Failure.md](./01-Fundamentals/09-Single-Point-of-Failure.md)

### 2. Networking and communication
- [01-DNS.md](./02-Networking/01-DNS.md)
- [02-HTTP-HTTPS.md](./02-Networking/02-HTTP-HTTPS.md)
- [03-TCP-vs-UDP.md](./02-Networking/03-TCP-vs-UDP.md)
- [04-REST-GraphQL-gRPC.md](./02-Networking/04-REST-GraphQL-gRPC.md)
- [05-Load-Balancers.md](./02-Networking/05-Load-Balancers.md)
- [06-Reverse-Proxies.md](./02-Networking/06-Reverse-Proxies.md)
- [07-API-Gateway.md](./02-Networking/07-API-Gateway.md)
- [08-WebSockets.md](./02-Networking/08-WebSockets.md)
- [09-CDN.md](./02-Networking/09-CDN.md)

### 3. Databases (P1)
- [01-SQL-vs-NoSQL.md](./03-Databases/01-SQL-vs-NoSQL.md)
- [02-Indexing.md](./03-Databases/02-Indexing.md)
- [03-Normalization-Denormalization.md](./03-Databases/03-Normalization-Denormalization.md)
- [04-Partitioning-Sharding.md](./03-Databases/04-Partitioning-Sharding.md)
- [05-Replication-Read-Replicas.md](./03-Databases/05-Replication-Read-Replicas.md)
- [06-Transactions-ACID.md](./03-Databases/06-Transactions-ACID.md)
- [07-Eventual-Consistency.md](./03-Databases/07-Eventual-Consistency.md)
- [08-Connection-Pooling.md](./03-Databases/08-Connection-Pooling.md)

### 4. Caching (P2)
- [01-Why-Caching.md](./04-Caching/01-Why-Caching.md)
- [02-Redis.md](./04-Caching/02-Redis.md)
- [03-Cache-Strategies.md](./04-Caching/03-Cache-Strategies.md)
- [04-Invalidation-Eviction.md](./04-Caching/04-Invalidation-Eviction.md)
- [05-Distributed-Caching.md](./04-Caching/05-Distributed-Caching.md)

### 5. Messaging and async (P4)
- [01-Message-Queues.md](./05-Messaging/01-Message-Queues.md)
- [02-Kafka-vs-RabbitMQ-vs-SQS.md](./05-Messaging/02-Kafka-vs-RabbitMQ-vs-SQS.md)
- [03-Pub-Sub-Event-Driven.md](./05-Messaging/03-Pub-Sub-Event-Driven.md)
- [04-DLQ-Retries-Idempotency.md](./05-Messaging/04-DLQ-Retries-Idempotency.md)
- [05-Delivery-Guarantees.md](./05-Messaging/05-Delivery-Guarantees.md)

### 6. Scalability and distributed systems (P3)
- [01-Stateless-Horizontal-Scaling.md](./06-Scalability/01-Stateless-Horizontal-Scaling.md)
- [02-Distributed-Locking.md](./06-Scalability/02-Distributed-Locking.md)
- [03-Rate-Limiting.md](./06-Scalability/03-Rate-Limiting.md)
- [04-Circuit-Breakers.md](./06-Scalability/04-Circuit-Breakers.md)
- [05-Service-Discovery.md](./06-Scalability/05-Service-Discovery.md)
- [06-Microservices-vs-Monolith.md](./06-Scalability/06-Microservices-vs-Monolith.md)
- [07-Consistent-Hashing.md](./06-Scalability/07-Consistent-Hashing.md)
- [08-Distributed-Transactions-Saga.md](./06-Scalability/08-Distributed-Transactions-Saga.md)

### 7. AWS architecture
- [01-AWS-Services-Practical.md](./07-AWS/01-AWS-Services-Practical.md)
- [02-Production-Architecture-Patterns.md](./07-AWS/02-Production-Architecture-Patterns.md)

### 8. High-level design problems (P5)
- [01-URL-Shortener.md](./08-HLD-Problems/01-URL-Shortener.md)
- [02-WhatsApp-Chat.md](./08-HLD-Problems/02-WhatsApp-Chat.md)
- [03-YouTube-Video-Streaming.md](./08-HLD-Problems/03-YouTube-Video-Streaming.md)
- [04-Instagram-Feed.md](./08-HLD-Problems/04-Instagram-Feed.md)
- [05-Notification-System.md](./08-HLD-Problems/05-Notification-System.md)
- [06-Rate-Limiter.md](./08-HLD-Problems/06-Rate-Limiter.md)
- [07-Ride-Booking.md](./08-HLD-Problems/07-Ride-Booking.md)
- [08-Ecommerce.md](./08-HLD-Problems/08-Ecommerce.md)
- [09-File-Storage.md](./08-HLD-Problems/09-File-Storage.md)
- [10-Food-Delivery.md](./08-HLD-Problems/10-Food-Delivery.md)

### 9. Mock interviews
- [01-How-Mocks-Work.md](./09-Mock-Interviews/01-How-Mocks-Work.md)

### Existing lecture-style notes (legacy)
- [1 Intro.md](./1%20Intro.md)
- [2 Scaling.md](./2%20Scaling.md)
- [3 Database Failover Strategies.md](./3%20Database%20Failover%20Strategies.md)
- [4 Scalable Database Design: Sharding, High Availability, and Data Modeling.md](./4%20Scalable%20Database%20Design:%20Sharding,%20High%20Availability,%20and%20Data%20Modeling.md)

---

## Senior interview mantra

> Clarify → Estimate → APIs → Data → Architecture → Flow → Bottlenecks → Failures → Trade-offs
