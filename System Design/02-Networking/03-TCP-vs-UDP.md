# TCP vs UDP

## 1. Simple Explanation

**TCP:** reliable, ordered, connection-based byte stream. Retries lost packets.
**UDP:** lightweight, connectionless datagrams. No built-in reliability/ordering.

HTTP/HTTPS typically runs on TCP. Video streaming often uses UDP-based protocols for speed.

## 2. Practical Backend Example

Your NestJS REST API uses TCP (via HTTP). A multiplayer game position update or live video might use UDP to avoid stalling on lost packets.

## 3. How It Works (step by step)

TCP:
1. Handshake (SYN/SYN-ACK/ACK).
2. Send data with sequence numbers/acks.
3. Retransmit losses; ensure order.

UDP:
1. Send packet to destination.
2. No guarantee of delivery/order.
3. App may add reliability if needed.

## 5. Why Do We Need It?

Choose reliability vs speed/latency characteristics based on workload.

## 6. Alternatives & Trade-offs

| | TCP | UDP |
|--|-----|-----|
| Reliability | Yes | No (by default) |
| Ordering | Yes | No |
| Latency overhead | Higher | Lower |
| Typical use | APIs, DB, SSH | Gaming, VoIP, some video |

## 7. Interview Revision Notes

- APIs = TCP.
- UDP when losing some data is better than waiting.
- QUIC/HTTP3 uses UDP under the hood but adds reliability at another layer.

## 8. Common Interview Q&A

**Q1. TCP vs UDP in one line?**
TCP reliable ordered; UDP fast best-effort.

**Q2. Why not UDP for bank transfers API?**
Need reliable delivery and standard HTTP/TLS stacks.

## 9. Practical Production Scenario

Live score push over TCP websocket stalls behind a lost packet retransmission; for some media paths UDP-based transport reduces freeze.

## 10. Self-Check Questions

1. Does TCP guarantee delivery?
2. When prefer UDP?
3. What protocol does typical REST use?
