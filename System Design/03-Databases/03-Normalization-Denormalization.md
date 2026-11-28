# Normalization and Denormalization

## 1. Simple Explanation

**Normalization:** store data without duplication; use related tables (users, orders, items). Good for write integrity.
**Denormalization:** intentionally duplicate data to make reads faster/simpler (store `user_name` on order).

## 2. Practical Backend Example

Normalized: `orders.user_id` joins to `users.name`.
Denormalized: `orders.user_name` copied at purchase time for invoice history even if user later changes name.

## 3. How It Works (step by step)

1. Start normalized for correctness.
2. Measure slow joins/reads.
3. Denormalize specific hot read paths.
4. Update strategy for duplicated fields (sync on write, async, or immutable snapshot).

## 5. Why Do We Need It?

At scale, joins across huge tables get expensive; feeds often denormalize.

## 6. Alternatives & Trade-offs

| | Normalized | Denormalized |
|--|------------|--------------|
| Writes | Cleaner | More update fan-out |
| Reads | More joins | Faster single lookup |
| Consistency | Easier | Risk of stale copies |

## 7. Interview Revision Notes

- Normalize by default.
- Denormalize for known hot reads.
- Snapshot fields that must not change historically.

## 8. Common Interview Q&A

**Q1. Why denormalize?**
Reduce joins / speed reads for specific access patterns.

**Q2. Risk?**
Inconsistent duplicated data if updates aren't handled.

## 9. Practical Production Scenario

Feed service stores precomputed post cards with author name/avatar to avoid join storms at read time.

## 10. Self-Check Questions

1. Define normalization simply.
2. When keep historical price denormalized on order line?
3. How do you update denormalized avatar URLs?
