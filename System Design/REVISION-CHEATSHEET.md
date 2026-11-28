# System Design — One-Page Revision Cheat Sheet (7+ YOE)

## Interview sequence
Clarify FR → NFR → Estimate → APIs → Schema → Architecture → Flow → Bottlenecks → Scale → Failures/Trade-offs

## Numbers
- 1 day ≈ 10^5 seconds
- Avg QPS ≈ events/day ÷ 10^5; peak ≈ 3–10× avg
- 1B × 1KB ≈ 1TB

## Scaling
- Vertical = bigger box | Horizontal = more boxes
- Stateless API + LB unlocks horizontal scale
- Scale reads: cache + replicas | Scale writes: partition/shard

## CAP / consistency
- During partition: pick C or A (P required)
- Money/inventory → stronger consistency
- Likes/feeds/counters → often eventual
- Read-your-writes: read primary after write

## Databases
- Postgres: relationships, ACID, complex queries
- DynamoDB: known access patterns, massive scale
- Index for real queries; too many indexes hurt writes
- Replicas ≠ write scale
- Pool connections; `instances × pool < max_conn`

## Caching
- Cache-aside + TTL + invalidate on write (default answer)
- Redis shared across instances
- Stampede: jitter, singleflight
- Cache ≠ source of truth

## Messaging
- Queue: buffer, decouple, async side effects
- SQS simple tasks | SNS fanout | Kafka event log/replay
- Assume at-least-once → idempotent consumers
- Retries + DLQ + alarms

## Reliability
- Timeouts on every dependency
- Circuit breaker + bulkhead
- Idempotency keys on payments/webhooks
- Remove SPOFs (multi-AZ, health checks)

## AWS quick map
- ALB + ECS API | RDS data | ElastiCache Redis
- S3 + CloudFront media | SNS/SQS async | CloudWatch observe
- API Gateway for managed API edge / Lambda

## Senior phrases
- \"Assumptions first…\"
- \"I'd start simple, then scale the bottleneck…\"
- \"Trade-off is X vs Y…\"
- \"Failure mode: … mitigated by …\"
- \"Consistency need here is … because …\"
