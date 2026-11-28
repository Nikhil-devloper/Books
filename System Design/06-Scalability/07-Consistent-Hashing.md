# Consistent Hashing

## 1. Simple Explanation

**Consistent hashing** places data/servers on a ring so when nodes are added/removed, only a small fraction of keys move — not almost all keys.

## 2. Practical Backend Example

Redis Cluster / CDN caches / sharded stores use consistent hashing so scaling from 4→5 nodes doesn't reshuffle everything.

## 3. How It Works (step by step)

1. Hash nodes onto a ring.
2. Hash key; walk ring to nearest node.
3. On node add/remove, only nearby keys remap.
4. Virtual nodes improve balance.

## 5. Why Do We Need It?

Minimize data movement during cluster rebalancing.

## 6. Alternatives & Trade-offs

| Modulo hashing `key % N` | Consistent hashing |
|--------------------------|--------------------|
| N change moves most keys | Few keys move |

## 7. Interview Revision Notes

- Know why `key % N` is painful when N changes.
- Used in caches, shards, CDNs.

## 8. Common Interview Q&A

**Q1. What problem does consistent hashing solve?**
Stable key→node mapping when cluster size changes.

**Q2. Virtual nodes?**
Multiple positions per server for better balance.

## 9. Practical Production Scenario

Cache cluster +1 node with modulo → mass miss storm. Consistent hashing limits miss storm.

## 10. Self-Check Questions

1. Why modulo fails on scale-out?
2. What is a hash ring?
3. Where used in AWS-ish stacks?
