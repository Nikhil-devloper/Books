# Database Failover Strategies

## 1. What is Database Failover?

Failover is the process of switching to a backup database when the primary database fails.

**Why do we need it?**

- A single database can become a Single Point of Failure (SPOF).
- If the database crashes, the application may stop working.
- A backup database helps keep the application available.

There are four common failover strategies:

1. Cold Standby
2. Warm Standby
3. Hot Standby
4. Multi-Primary

---

## 2. Cold Standby

**What is it?**

A backup database is available, but it is not continuously synchronized with the primary database.

**How does it work?**

1. The primary database handles all requests.
2. Periodic backups are taken.
3. Backups are stored somewhere safe, such as Amazon S3.
4. If the primary database crashes, restore the latest backup to the standby database.
5. Redirect application traffic to the new database.

**Example:**

- Backup taken at 10 AM.
- Primary database crashes at 12 PM.
- Restore the 10 AM backup.
- Data created between 10 AM and 12 PM may be lost.

**Advantages:**
- Cheap.
- Simple to implement.

**Disadvantages:**
- High downtime because backup restoration takes time.
- Data created after the last backup can be lost.

**Use case:** Internal applications where some downtime and data loss are acceptable.

**Interview one-liner:** Cold standby uses periodic backups to recover a failed database, but recovery takes time and recent data may be lost.

---

## 3. Warm Standby

**What is it?**

A secondary database continuously receives data from the primary database through replication.

**What is Replication?**

Replication means continuously copying data from one database to another.

**How does it work?**

1. The primary database handles application requests.
2. A secondary database stays ready as a backup.
3. Data is continuously replicated from the primary to the secondary.
4. If the primary database crashes, redirect traffic to the secondary.
5. The secondary becomes the new primary database.

**Example:**

When a new customer is created in the primary database, that record is automatically copied to the secondary database.

If the primary crashes, the secondary already has a copy of most of the data.

**Advantages:**
- Less downtime than cold standby.
- Less data loss.
- Faster recovery.

**Disadvantages:**
- Replication may have a small delay.
- Some recent data may still be lost.
- Additional infrastructure cost.

**Interview one-liner:** Warm standby continuously replicates data to a backup database, allowing faster failover with minimal downtime and possible small data loss.

---

## 4. Hot Standby

**What is it?**

A secondary database is continuously synchronized with the primary database in real time and can also serve read requests.

**How does it work?**

1. The primary database handles read and write requests.
2. Data is replicated to the standby database in real time.
3. The standby database can also handle read requests.
4. If the primary database crashes, the standby takes over read and write operations.

**Example:**

In an e-commerce application:

- Primary database handles reads and writes.
- Standby database handles additional read requests.
- Both databases have synchronized data.
- If the primary crashes, the standby takes over.

**Advantages:**
- Very little downtime.
- Faster failover.
- Standby database can help distribute read traffic.

**Disadvantages:**
- Higher infrastructure cost.
- Replication and failover need proper management.

**Interview one-liner:** Hot standby maintains a real-time synchronized backup database that can serve reads and take over writes when the primary fails.
---

Continuous replication
Replication runs continuously, but changes may take a few seconds to reach the replica.

Real-time replication
Changes are propagated with very low latency.

## 5. Multi-Primary Database

**What is it?**

Multiple database nodes can independently handle both read and write requests.

Unlike warm and hot standby, we are not relying on a single primary database to handle writes.

**How does it work?**

1. Multiple primary databases are available.
2. Application writes data to multiple database nodes.
3. Both databases can handle read and write requests.
4. If one database crashes, traffic is redirected to the remaining database.
5. When the failed database recovers, it must synchronize its data before returning to service.

**Example:**

Suppose we have two database nodes:

- Database A
- Database B

Both can accept read and write requests.

If Database A crashes, the application can continue using Database B.

**Advantages:**
- Better availability.
- Multiple databases handle read and write requests.
- Moves closer to horizontal database scaling.

**Disadvantages:**
- Write conflicts can happen.
- Data synchronization becomes complicated.
- Conflict resolution is required.
- Recovery and resynchronization need careful handling.

**Interview one-liner:** Multi-primary allows multiple database nodes to accept reads and writes simultaneously, improving availability and scaling but introducing data consistency challenges.

---

## 6. Comparison of Failover Strategies

| Feature | Cold Standby | Warm Standby | Hot Standby | Multi-Primary |
|---|---|---|---|---|
| Data synchronization | Periodic backup | Continuous replication | Real-time replication | Multiple writable nodes |
| Standby reads | No | No | Yes | Yes |
| Standby writes | After failover | After failover | After failover | Yes |
| Downtime | High | Low | Very low | Low |
| Data loss risk | High | Low | Very low | Depends on replication |
| Complexity | Low | Medium | Medium | High |
| Cost | Low | Medium | Higher | Higher |

*Actual downtime and data loss depend on the replication configuration and failover implementation.*

---

## 7. Interview Questions with One-Line Answers

**Q1. What is database failover?**

Switching application traffic from a failed primary database to a backup database.

**Q2. Why is a single database a Single Point of Failure?**

Because if that database crashes, the entire application may lose access to its data.

**Q3. What is cold standby?**

A backup database is restored from periodic backups when the primary database fails.

**Q4. What are the disadvantages of cold standby?**

High downtime and potential data loss because recovery depends on the last available backup.

**Q5. What is database replication?**

Continuously copying data from one database to another to maintain a backup copy.

**Q6. What is the difference between cold and warm standby?**

Cold standby uses periodic backups, whereas warm standby continuously replicates data.

**Q7. What is the difference between warm and hot standby?**

Warm standby maintains a replicated backup, whereas hot standby can also serve read requests.

**Q8. Why can hot standby handle read requests?**

Because its data is continuously synchronized with the primary database, allowing it to serve reads.

**Q9. What is a multi-primary database?**

Multiple database nodes can independently accept read and write requests.

**Q10. What are the challenges of multi-primary databases?**

Write conflicts, data consistency, synchronization and conflict resolution.

---

# 8. Quick Revision – Read Before Interviews

### Four Database Failover Strategies

| Strategy | Remember This |
|---|---|
| Cold Standby | Periodic backup → Restore → Redirect traffic. |
| Warm Standby | Continuous replication → Failover when primary crashes. |
| Hot Standby | Real-time replication → Supports reads and takes over writes. |
| Multi-Primary | Multiple databases accept reads and writes simultaneously. |

### Important Concepts

- **Failover:** Switching to a backup database when the primary fails.
- **SPOF:** A single component whose failure can bring down the entire system.
- **Replication:** Copying data from one database to another.
- **Cold vs Warm:** Periodic backup vs Continuous replication.
- **Warm vs Hot:** Backup standby vs Standby that can also serve reads.
- **Multi-Primary:** Multiple writable database nodes.
- **Main Multi-Primary Challenge:** Write conflicts and data consistency.

### Interview Memory Triggers

1. Database crashes → Failover.
2. Periodic backup → Cold standby.
3. Continuous replication → Warm standby.
4. Real-time replication + Read traffic → Hot standby.
5. Multiple writable databases → Multi-primary.
6. Multiple nodes writing same data → Conflict resolution.
