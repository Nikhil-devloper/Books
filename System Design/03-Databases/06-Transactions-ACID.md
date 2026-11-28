# Transactions and ACID

## 1. Simple Explanation

A **transaction** groups multiple DB operations that succeed or fail as one unit.

**ACID:**
- **Atomicity:** all or nothing.
- **Consistency:** DB rules/invariants hold after commit.
- **Isolation:** concurrent transactions don't badly interfere (levels vary).
- **Durability:** committed data survives crashes.

## 2. Practical Backend Example

Place order: insert order + debit inventory + insert payment row. If payment insert fails, roll back all so you don't sell without payment record.

## 3. How It Works (step by step)

1. BEGIN
2. Execute statements.
3. On success COMMIT (durable).
4. On error ROLLBACK.
5. Isolation level controls what concurrent txs see.

## 5. Why Do We Need It?

Prevents partial updates that corrupt business state.

## 6. Alternatives & Trade-offs

| Isolation level | Phenomena prevented | Cost |
|-----------------|---------------------|------|
| Read committed | Dirty reads | Default common |
| Repeatable read | + non-repeatable | More locking/MVCC cost |
| Serializable | Strongest | Highest contention |

## 7. Interview Revision Notes

- Use transactions for multi-step correctness.
- Keep transactions short.
- Distributed multi-service transactions need sagas (not one giant ACID).

## 8. Common Interview Q&A

**Q1. Explain ACID briefly.**
Atomic, Consistent, Isolated, Durable — safe transactional properties.

**Q2. Why short transactions?**
Long txs hold locks/versions → latency and deadlocks.

## 9. Practical Production Scenario

Two users buy last item. Without proper transaction/locking, both succeed. With row lock/`SELECT FOR UPDATE`/conditional update, only one wins.

## 10. Self-Check Questions

1. Meaning of atomicity?
2. Dirty read example?
3. Why not one ACID tx across Order + Email + Inventory microservices?
