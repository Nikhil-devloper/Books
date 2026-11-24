# Scalable Database Design: Sharding, High Availability, and Data Modeling

## 1. Why do we need a scalable database?

As an application grows, a single database server can become a
bottleneck because it has limited:

-   CPU and memory
-   Storage capacity
-   Read/write throughput
-   Ability to handle failures without downtime

There are two common ways to scale:

-   **Vertical scaling:** Increase the resources of one database server.
-   **Horizontal scaling:** Distribute data and workload across multiple
    database servers.

This chapter focuses on horizontal database scaling.

------------------------------------------------------------------------

## 2. Database Sharding

### What is sharding?

**Sharding is the process of splitting a database into smaller
horizontal partitions and storing those partitions on different database
servers.**

Each partition is called a **shard**.

For example, imagine a users table with 9 million users:

  Shard     User IDs
  --------- --------------
  Shard 1   1--3 million
  Shard 2   3--6 million
  Shard 3   6--9 million

Instead of one database server handling all 9 million users, three
shards share the workload.

```{=html}
<figure>
```
`<img src="https://dummyimage.com/900x360/f2f2f2/333333&text=Application+%E2%86%92+Query+Router+%E2%86%92+Shard+1+%7C+Shard+2+%7C+Shard+3" alt="Conceptual sharding diagram" />`{=html}
```{=html}
</figure>
```
### How does a request reach the correct shard?

A **query router** determines which shard contains the required data.

A common approach is to use a **shard key** and a partitioning function.

Example:

``` text
shard = hash(userId) % numberOfShards
```

If the application requests data for `userId = 123`, the routing logic
calculates the target shard and sends the request there.

The exact algorithm depends on the database and its partitioning
strategy.

### What is a shard key?

A **shard key** is the field (or combination of fields) used to
distribute records across shards.

Examples:

-   `userId`
-   `customerId`
-   `orderId`
-   A combination such as `region + customerId`

A good shard key should distribute workload evenly and support the
application's common query patterns.

### Benefits of sharding

1.  **Horizontal scalability:** Add shards to increase database
    capacity.
2.  **Higher throughput:** Multiple database servers can process
    requests in parallel.
3.  **Larger storage capacity:** Data is distributed across servers.
4.  **Workload distribution:** No single database server needs to handle
    the entire dataset.

### Challenges of sharding

1.  **Cross-shard queries:** Joining or querying data across multiple
    shards is more complicated and can be slower.
2.  **Resharding:** Adding or removing shards may require data
    redistribution.
3.  **Hotspots:** Some shards may receive much more traffic than others.
4.  **Operational complexity:** Monitoring, backups, routing, and
    failure handling become more complicated.

### What is a hotspot?

A **hotspot** occurs when one shard receives disproportionately high
traffic compared with other shards.

Example:

Imagine a movie database. A popular actor's profile may receive millions
of requests, while profiles of less-known actors receive very few.

If all data for that popular actor is stored on one shard, that shard
can become overloaded.

This is sometimes called the **celebrity problem**.

Possible approaches include:

-   Choose a shard key that distributes traffic more evenly.
-   Monitor traffic and storage per shard.
-   Use caching for extremely popular records.
-   Split or redistribute heavily accessed data when supported by the
    database.

### What is resharding?

**Resharding is redistributing data across shards when the existing
partitioning arrangement needs to change.**

Example:

Initially:

``` text
Shard 1: Users A–F
Shard 2: Users G–M
```

After adding another shard, data may need to be redistributed:

``` text
Shard 1: Users A–D
Shard 2: Users E–H
Shard 3: Users I–M
```

The database must move data safely while minimizing downtime and
maintaining correct routing.

### Interview answer: What is sharding?

Sharding is horizontal database partitioning where data is split across
multiple database servers to improve scalability and distribute
read/write workload.

------------------------------------------------------------------------

## 3. Database Replication and High Availability

Sharding improves scalability, but it introduces another concern: **What
happens if a shard's database server fails?**

A common solution is replication.

### What is replication?

**Replication means maintaining copies of data on multiple database
servers.**

For example:

``` text
Shard 1
  ├── Primary
  ├── Secondary 1
  └── Secondary 2
```

The primary generally handles writes, and changes are replicated to
secondary nodes.

If the primary fails, an eligible secondary can be elected as the new
primary.

### Sharding vs. Replication

  -----------------------------------------------------------------------
  Sharding                            Replication
  ----------------------------------- -----------------------------------
  Splits data across servers          Copies data across servers

  Primarily improves scalability      Primarily improves availability and
                                      redundancy

  Each shard holds a portion of the   Replicas hold copies of the same
  dataset                             shard's data

  Helps distribute workload           Helps recover from node failure and
                                      can support read scaling
  -----------------------------------------------------------------------

**Remember:** Sharding and replication solve different problems. A
production database can use both together.

------------------------------------------------------------------------

## 4. MongoDB Architecture: Sharding and Replica Sets

MongoDB combines sharding and replication.

### Main components

#### 1. Application servers

These run the application and send database requests.

#### 2. `mongos` --- Query Router

`mongos` routes application requests to the appropriate shard.

It uses the cluster's metadata to determine where data is located and
which shard should receive a request.

#### 3. Shards

A shard stores a portion of the overall dataset.

In a typical MongoDB sharded cluster, each shard is backed by a
**replica set**.

#### 4. Replica set

A replica set is a group of MongoDB processes that maintain the same
dataset.

It typically contains:

-   **Primary:** Receives writes.
-   **Secondaries:** Replicate data from the primary and may serve
    eligible reads, depending on read preference.

If the primary becomes unavailable, eligible voting members can elect a
new primary.

#### 5. Config servers

Config servers store cluster metadata, including information about the
sharded cluster's data distribution.

A production sharded cluster uses a **config server replica set** so
that this metadata is also highly available.

### Simplified MongoDB request flow

1.  The application sends a request to `mongos`.
2.  `mongos` checks cluster metadata to identify the relevant shard.
3.  The request is routed to that shard.
4.  The shard's primary handles a write.
5.  The change is replicated to secondary members.
6.  If the primary fails, the replica set can elect a new primary.

### Why are multiple replica-set members used?

They provide redundancy and allow an election if the primary fails.

A replica set commonly has three voting members. With three voting
members, a majority is two, so the remaining two can still form a
majority if one member fails.

**Important:** Three members are a common deployment pattern, not a
universal minimum for every possible MongoDB replica-set configuration.

### Interview answer: How does MongoDB achieve high availability?

MongoDB uses replica sets. A primary handles writes, secondaries
replicate the data, and eligible members can automatically elect a new
primary if the existing primary fails.

### Interview answer: What is the role of `mongos`?

`mongos` is the query router in a MongoDB sharded cluster. It routes
application requests to the correct shard using cluster metadata.

------------------------------------------------------------------------

## 5. Cassandra Architecture: Peer-to-Peer Design

MongoDB commonly uses a primary-secondary replica-set model. Cassandra
uses a different architecture.

### How does Cassandra work?

Cassandra uses a **peer-to-peer architecture**.

-   Data is partitioned across nodes.
-   Data is replicated to multiple nodes for redundancy.
-   Any node can receive a client request.
-   The receiving node acts as a **coordinator** for that request.
-   There is no single cluster-wide primary node.

### What is a coordinator node?

The coordinator is the node that receives a client request and
coordinates the operation with the nodes responsible for the relevant
data.

It does not mean that this node permanently owns or becomes the primary
for that data.

### Why is Cassandra highly available?

Because there is no single cluster-wide primary that all requests must
pass through.

If one node fails, clients can contact another available node, and the
cluster can continue serving requests if the required replicas are
available.

### What is eventual consistency?

**Eventual consistency means that replicas may temporarily contain
different values, but they are expected to converge if updates stop and
replication succeeds.**

Example:

1.  A user updates their profile.
2.  The write is accepted by the database.
3.  The updated value has not yet reached every replica.
4.  A read from another replica may temporarily return the older value.
5.  After replication catches up, replicas return the updated value.

### Cassandra trade-off

  -----------------------------------------------------------------------
  Advantage                           Trade-off
  ----------------------------------- -----------------------------------
  No single cluster-wide primary      Replicas may temporarily differ

  High availability                   Reads may not immediately observe
                                      the latest write, depending on
                                      consistency settings

  Horizontal scalability              Distributed coordination and data
                                      modeling are more complex
  -----------------------------------------------------------------------

**Important:** Cassandra does not always mean eventual consistency for
every read. Its consistency level controls how many replicas must
respond to a read or write. With suitable consistency levels, stronger
consistency guarantees can be achieved for particular operations, with
availability and latency trade-offs.

### MongoDB vs. Cassandra

  -----------------------------------------------------------------------
  Area                    MongoDB                 Cassandra
  ----------------------- ----------------------- -----------------------
  Architecture            Sharded clusters can    Peer-to-peer
                          use replica sets        

  Request routing         `mongos` routes         Any node can act as
                          requests to shards      coordinator

  Replication model       Primary-secondary       Data replicated across
                          within a replica set    peer nodes

  Primary role            Each replica set has a  No single cluster-wide
                          primary                 primary

  Failure handling        Replica-set election    Other nodes can
                                                  coordinate requests;
                                                  replicas provide
                                                  redundancy

  Consistency             Depends on read/write   Tunable consistency
                          concern and             levels
                          configuration           
  -----------------------------------------------------------------------

### Interview answer: How is Cassandra different from MongoDB?

MongoDB uses primary-secondary replica sets within its shards, while
Cassandra uses a peer-to-peer architecture where any node can coordinate
requests. Cassandra offers tunable consistency levels.

------------------------------------------------------------------------

## 6. NoSQL Databases

### What is NoSQL?

NoSQL refers to database systems that do not follow the traditional
relational database model as their primary data model.

Common NoSQL database categories include:

-   **Key-value:** Redis, Amazon DynamoDB
-   **Document:** MongoDB
-   **Wide-column:** Apache Cassandra, HBase
-   **Graph:** Neo4j

NoSQL databases are often selected for flexible data models, horizontal
scalability, and specific access patterns.

### Does NoSQL mean that SQL cannot be used?

No. The term NoSQL does not simply mean that SQL is impossible.

However, NoSQL databases do not all use SQL as their query language:

-   MongoDB uses its document query API.
-   Cassandra uses CQL (Cassandra Query Language), which resembles SQL.
-   DynamoDB uses its own API and expression-based query model.

The important distinction is the data model and access pattern, not
merely whether the query language looks like SQL.

### Why are key-value lookups important?

Distributed databases work best when the application knows which
partition contains the required data.

A direct lookup using a partition key can route a request efficiently.

Example:

``` text
Get customer with customerId = 123
```

The database can use the partition key to locate the relevant data.

Complex joins across partitions can require additional coordination,
network calls, or data movement.

**Design principle:** Model data around the queries the application
needs to execute, and prefer efficient partition-key lookups when
designing for large-scale distributed databases.

### Schema flexibility

Some NoSQL databases allow flexible or evolving record structures.

For example, two MongoDB documents in the same collection can contain
different fields.

This does not mean schema design is unnecessary. Applications still need
validation, consistent data contracts, and a clear data model.

### Examples

  Database           Type
  ------------------ ---------------------------------
  MongoDB            Document database
  Apache Cassandra   Wide-column database
  Amazon DynamoDB    Key-value and document database
  Apache HBase       Wide-column database

### Interview answer: Why do distributed databases prefer key-value lookups?

Key-value lookups can target a known partition directly, reducing
cross-shard coordination and making data access more efficient.

------------------------------------------------------------------------

## 7. Database Normalization

Before understanding denormalization, understand normalization.

### What is normalization?

**Normalization organizes data into separate related tables to reduce
duplication and improve data consistency.**

Consider a restaurant reservation system.

A reservation contains:

  Reservation ID   Customer ID   Time Slot
  ---------------- ------------- -----------
  R1               C101          7 PM
  R2               C101          8 PM

Customer details are stored separately:

  Customer ID   Name    Phone
  ------------- ------- ------------
  C101          Rahul   9876543210

The reservation table stores `Customer ID` rather than repeating the
customer's name and phone number in every reservation.

### How are the tables connected?

-   `Customer ID` is the primary key in the customer table.
-   `Customer ID` in the reservation table is a foreign key.
-   A JOIN combines reservation data with customer data.

Example SQL:

``` sql
SELECT
    r.reservation_id,
    r.time_slot,
    c.name,
    c.phone
FROM reservations r
JOIN customers c
    ON r.customer_id = c.customer_id;
```

### Advantages of normalization

1.  **Less duplication:** Customer details are stored in one place.
2.  **Easier updates:** Change a customer's phone number once.
3.  **Better consistency:** Multiple reservations refer to the same
    customer record.
4.  **Storage efficiency:** Repeated customer details are not stored in
    every reservation row.

### Disadvantages

-   Retrieving related information may require JOINs.
-   Complex queries can become expensive at large scale, depending on
    indexes, data volume, and database architecture.

### Interview answer: What is normalization?

Normalization organizes data into related tables to minimize duplication
and improve data consistency.

------------------------------------------------------------------------

## 8. Database Denormalization

### What is denormalization?

**Denormalization intentionally duplicates data to reduce reads, JOINs,
or additional database requests.**

Using the same reservation example:

  --------------------------------------------------------------------------
  Reservation ID Customer ID    Customer Name  Customer Phone Time Slot
  -------------- -------------- -------------- -------------- --------------
  R1             C101           Rahul          9876543210     7 PM

  R2             C101           Rahul          9876543210     8 PM
  --------------------------------------------------------------------------

Customer details are copied into each reservation record.

Now, the application can retrieve the reservation and customer details
with one lookup.

### Advantages of denormalization

1.  **Faster reads for common access patterns:** Required information
    can be retrieved together.
2.  **Fewer JOINs:** Related data is already available in the same
    record or data structure.
3.  **Fewer database round trips:** The application may need fewer
    queries.
4.  **Useful for read-heavy workloads:** Especially when the same data
    is frequently read together.

### Disadvantages of denormalization

1.  **Data duplication:** The same customer details are stored multiple
    times.
2.  **Harder updates:** A customer's phone number may need to be updated
    in multiple records.
3.  **Risk of inconsistency:** Some copies may be updated while others
    remain stale.
4.  **More storage:** Duplicated fields consume additional space.

### Normalization vs. Denormalization

  -----------------------------------------------------------------------
  Area                    Normalization           Denormalization
  ----------------------- ----------------------- -----------------------
  Data duplication        Minimized               Intentionally increased

  Read performance        May require JOINs       Often faster for known
                                                  access patterns

  Write/update complexity Generally simpler       More complex

  Data consistency        Easier to maintain      Requires
                                                  synchronization

  Storage                 More efficient          Uses more storage

  Best suited for         Write consistency and   Read-heavy,
                          maintainability         performance-sensitive
                                                  access patterns
  -----------------------------------------------------------------------

### When should you denormalize?

Do not denormalize everything by default.

A sensible approach:

1.  Start with a normalized data model.
2.  Identify the application's most frequent and expensive queries.
3.  Measure whether JOINs or multiple database requests are actually a
    bottleneck.
4.  Denormalize only where it improves an important access pattern.
5.  Define how duplicated data will be updated and kept consistent.

### Example: When denormalization makes sense

Suppose a reservation details page is requested millions of times per
day.

Every request needs:

-   Reservation details
-   Customer name
-   Customer phone number

If retrieving these fields requires expensive cross-partition
operations, storing a copy of the required customer details with the
reservation may improve read performance.

However, if the customer changes their phone number, the application
needs a strategy to update the duplicated values.

### Interview answer: What is denormalization?

Denormalization intentionally duplicates related data to reduce JOINs
and database round trips, improving read performance at the cost of
storage and update complexity.

### Interview answer: Should we always denormalize in NoSQL?

No. Start with the access patterns and choose the simplest model that
meets performance requirements. Denormalize when measured
read-performance needs justify the duplication and synchronization cost.

------------------------------------------------------------------------

## 9. How to Approach These Concepts in a System Design Interview

When designing a database, follow this thought process:

1.  **Understand the workload:** Is the system read-heavy, write-heavy,
    or mixed?
2.  **Choose the data model:** Relational or NoSQL, based on
    relationships, consistency, queries, and scale.
3.  **Design the partitioning strategy:** If horizontal scaling is
    needed, choose a suitable shard/partition key.
4.  **Plan for failures:** Use replication and appropriate failover
    mechanisms.
5.  **Design around access patterns:** Prefer efficient queries that
    target known partitions.
6.  **Consider denormalization:** Duplicate data only when it solves a
    demonstrated performance problem.
7.  **Consider operational challenges:** Hotspots, resharding, backups,
    monitoring, and consistency.

### A practical example

Imagine designing a large e-commerce application.

-   Orders can be partitioned by `customerId` or another key chosen from
    the access patterns.
-   Replication can protect data against individual node failures.
-   Order details may include a snapshot of product name and price so
    historical orders do not change when the product catalog changes.
-   Frequently accessed product information may be cached.
-   Cross-shard queries should be considered carefully.

The exact design depends on query patterns, consistency requirements,
traffic distribution, and operational constraints.

------------------------------------------------------------------------

# Quick Revision --- One-Liners

  -----------------------------------------------------------------------
  Topic                               One-line answer
  ----------------------------------- -----------------------------------
  Sharding                            Splitting a database horizontally
                                      across multiple servers to scale
                                      capacity and workload.

  Shard                               A horizontal partition of a
                                      database.

  Shard key                           The field used to decide how
                                      records are distributed across
                                      shards.

  Query router                        Routes a database request to the
                                      shard that owns the required data.

  Replication                         Maintaining copies of data across
                                      multiple database nodes.

  Primary                             The replica-set member that handles
                                      writes.

  Secondary                           A replica-set member that copies
                                      data from the primary.

  Replica set                         A group of database nodes
                                      maintaining the same dataset for
                                      redundancy and failover.

  High availability                   Keeping a system operational
                                      despite individual component
                                      failures.

  `mongos`                            MongoDB's query router that directs
                                      requests to the correct shard.

  Config server                       Stores metadata about a MongoDB
                                      sharded cluster.

  Cassandra coordinator               The node that receives a request
                                      and coordinates the operation with
                                      relevant replicas.

  Peer-to-peer architecture           An architecture where nodes operate
                                      as peers rather than depending on
                                      one cluster-wide primary.

  Eventual consistency                Replicas may temporarily differ but
                                      converge after updates propagate
                                      successfully.

  NoSQL                               A broad category of non-relational
                                      database systems with different
                                      data models.

  Hotspot                             A shard or node receiving
                                      disproportionately high traffic.

  Celebrity problem                   A hotspot caused by extremely
                                      popular data receiving much more
                                      traffic than other data.

  Resharding                          Redistributing data across shards
                                      when the partitioning arrangement
                                      changes.

  Key-value lookup                    Retrieving data using a known key,
                                      often allowing direct partition
                                      targeting.

  Normalization                       Structuring data into related
                                      tables to reduce duplication and
                                      improve consistency.

  Denormalization                     Duplicating data intentionally to
                                      improve read performance and reduce
                                      JOINs.

  Primary key                         A column or set of columns that
                                      uniquely identifies a row.

  Foreign key                         A column that references a key in
                                      another table.

  JOIN                                Combining related rows from
                                      multiple tables.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Interview Questions and Short Answers

**1. What is database sharding?**

Sharding splits a database into horizontal partitions across multiple
servers to improve scalability.

**2. Why do we need sharding?**

To distribute database workload and storage across multiple servers when
one server is insufficient.

**3. What is the difference between sharding and replication?**

Sharding splits data; replication copies data.

**4. What happens if a MongoDB primary node fails?**

Eligible secondary members can elect a new primary automatically.

**5. What is the role of `mongos`?**

It routes requests to the correct shard in a MongoDB sharded cluster.

**6. What is a MongoDB replica set?**

A group of MongoDB nodes that maintain the same dataset and provide
redundancy and automatic failover.

**7. How does Cassandra avoid a single cluster-wide primary?**

It uses a peer-to-peer architecture where any node can coordinate a
client request.

**8. What is eventual consistency?**

A read may temporarily return stale data, but replicas are expected to
converge after updates propagate.

**9. What is a hotspot?**

A database partition receiving significantly more traffic than other
partitions.

**10. What is resharding?**

Redistributing existing data across shards when the partitioning
arrangement changes.

**11. Why are cross-shard JOINs challenging?**

Related data may reside on different servers, requiring additional
network communication and coordination.

**12. What is normalization?**

Organizing data into related tables to minimize duplication and maintain
consistency.

**13. What is denormalization?**

Intentionally duplicating related data to reduce JOINs and improve read
performance.

**14. What is the main disadvantage of denormalization?**

Updates become more complex because duplicated values must remain
synchronized.

**15. Should we always use denormalization for scalability?**

No. Start with a simple normalized model and denormalize only when
actual query performance requires it.

------------------------------------------------------------------------

# Important Clarifications for Interview Accuracy

A few explanations in the lecture are simplified. Keep these
distinctions clear:

1.  **MongoDB primary vs. `mongos`:** `mongos` routes requests across
    shards. A replica-set primary handles writes for its own replica
    set; it is not the router for the entire shard cluster.
2.  **Three-node elections:** Three voting members are a common way to
    maintain a majority after one failure. The important concept is
    quorum/majority, not a universal rule that every cluster must have
    exactly three nodes.
3.  **Cassandra consistency:** Cassandra is not necessarily eventually
    consistent for every operation. Consistency levels determine how
    many replicas must acknowledge reads and writes.
4.  **NoSQL and SQL:** NoSQL does not mean every database uses SQL.
    MongoDB, Cassandra, and DynamoDB have different query interfaces.
5.  **Denormalization:** It is not automatically faster in every
    situation. The benefit depends on the actual access pattern,
    indexes, network calls, and database workload.

------------------------------------------------------------------------

## Final Mental Model

Remember the chapter using these five questions:

-   **Need more database capacity?** → Sharding.
-   **Need copies of data for failure recovery?** → Replication.
-   **Need to avoid a single primary dependency?** → Understand
    peer-to-peer designs such as Cassandra.
-   **Need less duplication and easier updates?** → Normalization.
-   **Need faster reads with fewer JOINs?** → Consider denormalization.

**The core interview skill is explaining why you chose a particular
database design and what trade-off you accepted.**
