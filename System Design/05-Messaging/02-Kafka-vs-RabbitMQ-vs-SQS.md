# Kafka vs RabbitMQ vs SQS

## 1. Simple Explanation

Three common messaging systems with different strengths:
- **SQS:** managed queue on AWS; simple, serverless-friendly.
- **RabbitMQ:** smart broker; routing, traditional message queues.
- **Kafka:** distributed log; high throughput event streams, replay, many consumers.

## 2. Practical Backend Example

Order emails: SQS + Lambda/ECS workers.
Complex routing rules inside datacenter: RabbitMQ.
Event stream of all user actions for many consumers (analytics, search index, fraud): Kafka.

## 3. How It Works (step by step)

SQS: push message → poll → process → delete.
Rabbit: exchange routes to queues by rules.
Kafka: append to partitioned topic log; consumers track offsets; can replay.

## 5. Why Do We Need It?

Pick based on replay needs, throughput, ops model, ordering, fan-out.

## 6. Alternatives & Trade-offs

| | SQS | RabbitMQ | Kafka |
|--|-----|----------|-------|
| Ops | Managed | You/managed service | Heavier |
| Replay | Limited | Not the model | Core feature |
| Throughput | High | High | Extremely high |
| Model | Queue | Queue/routing | Log/stream |

## 7. Interview Revision Notes

- Default AWS async: SQS (+ SNS for pub/sub fanout).
- Kafka when multiple consumers need the same event history.
- Don't introduce Kafka for a simple email queue.

## 8. Common Interview Q&A

**Q1. SQS vs Kafka?**
SQS = task queue. Kafka = durable event log with replay/fanout.

**Q2. When RabbitMQ?**
Flexible routing, traditional messaging patterns, existing RMQ expertise.

## 9. Practical Production Scenario

Team put Kafka in for password-reset emails. Ops cost high. Migrated to SQS — right-sized.

## 10. Self-Check Questions

1. Which supports replay best?
2. SNS+SQS pattern purpose?
3. Kafka partition role?
