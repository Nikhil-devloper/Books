# System Design Notes: Scalability

**Interview preparation notes \| AWS implementation mapping**

> **Source basis:** Frank Kane's lecture transcript on scalability,
> supplemented with AWS implementation examples and interview-oriented
> explanations. The AWS mappings below are practical examples, not
> claims made by the lecture.

## 1. What is scalability?

**Scalability** is the ability of a system to handle increasing
workloads without significant degradation in performance.

Imagine a cricket community application:

-   100 users access it: the application works smoothly.
-   10,000 users access it: response times may increase.
-   100,000 users access it: servers may become overloaded.
-   1 million users access it: parts of the system may become
    unavailable.

A scalable architecture handles growth by increasing capacity and
distributing workloads. The lecture focuses on progressing from a
single-server design to horizontal scaling.

## 2. Single-server architecture

**Architecture:**

``` text
Clients → Internet → One server (application + database)
```

Everything runs on one machine: the HTTP server, application, and often
the database.

### When it can be suitable

-   Small personal websites or low-traffic tools.
-   Applications with limited availability requirements.
-   Projects with tight budgets and low maintenance needs.

### Problems

-   The server is a **Single Point of Failure (SPOF)**. If it fails, the
    whole application can go down.
-   Application and database processes compete for CPU and memory.
-   The application and database cannot be scaled independently.
-   Recovery may require provisioning a new server, restoring backups,
    and changing DNS.

**AWS example:** EC2 running the application and a local database, with
EBS storage and snapshots for backup.

## 3. Separate application server and database

**Architecture:**

``` text
Clients → Internet → Application server → Separate database server
```

Moving the database to a separate machine allows the application and
database to use different resources and be scaled independently.

### Advantages

-   The database no longer competes directly with the web server for the
    same machine resources.
-   A data-intensive database can use a larger machine than the
    application server.
-   Application and database capacity can be adjusted independently.

### Remaining limitation

A single application server and a single database server are still
individual points of failure. If either fails, the website may become
unavailable.

**AWS example:** EC2 for the application, Amazon RDS for the database,
VPC for networking, and RDS automated backups.

## 4. Vertical scaling (scale up)

**Vertical scaling** means increasing the capacity of an existing
machine rather than adding more machines.

  Resource     Example before   Example after
  ---------- ---------------- ---------------
  CPU                  2 vCPU          8 vCPU
  Memory             4 GB RAM       32 GB RAM
  Storage               50 GB          500 GB

### Advantages

-   Relatively simple architecture.
-   Fewer machines and fewer components to maintain.
-   Can be a practical short-term response to increased load.

### Disadvantages

-   A machine can only grow so large: CPU, memory, and storage have
    upper limits.
-   Larger machines can become increasingly expensive.
-   A single machine remains a potential SPOF.

### AWS examples

-   **EC2:** Change to an instance type with more CPU and memory.
-   **RDS:** Increase the DB instance class.

The lecture emphasizes that vertical scaling is useful only up to a
limit.

## 5. Horizontal scaling (scale out)

**Horizontal scaling** means adding more machines and distributing
incoming work across them.

**Architecture:**

``` text
Clients → Load balancer → Multiple application servers → Database
```

Instead of relying on one increasingly powerful server, the system runs
a fleet of servers. As traffic grows, more servers can be added.

### Advantages

-   Capacity can grow by adding instances.
-   A load balancer can route traffic away from an unhealthy server.
-   With sufficient spare capacity, an individual application-server
    failure need not cause downtime.
-   The system can handle substantially larger workloads than a single
    server.

### Trade-offs

-   More infrastructure to operate and monitor.
-   Requires traffic distribution and health checks.
-   Application servers should be stateless or use shared state.
-   Distributed systems introduce additional complexity.

### AWS examples

  Concept               AWS service
  --------------------- ---------------------------------
  Load balancing        Application Load Balancer (ALB)
  Application servers   EC2 or ECS
  Automatic scaling     EC2 Auto Scaling
  Health checks         ALB health checks
  Database              Amazon RDS

Horizontal scaling is the central approach highlighted in the lecture
for large-scale system design. It is not literally unlimited: databases,
networks, storage, and other bottlenecks must also scale.

## 6. Vertical vs. horizontal scaling

  -----------------------------------------------------------------------
  Dimension               Vertical scaling        Horizontal scaling
  ----------------------- ----------------------- -----------------------
  Approach                Use a bigger machine    Add more machines

  Capacity                Increase                Distribute workload
                          CPU/RAM/storage on one  across hosts
                          host                    

  Complexity              Usually simpler         Usually more complex

  Limit                   Hardware/instance       Depends on architecture
                          ceiling                 and shared bottlenecks

  Failure risk            Single host may remain  Can tolerate an
                          a SPOF                  instance failure if
                                                  designed with
                                                  redundancy

  AWS example             Upgrade EC2/RDS         ALB + Auto Scaling
                          instance class          fleet
  -----------------------------------------------------------------------

## 7. Load balancers

A **load balancer** sits between clients and a group of application
servers. It distributes incoming requests among available servers.

### Common strategies

1.  **Round robin:** Send requests sequentially to each server.
2.  **Least connections:** Prefer the server with fewer active
    connections.
3.  **Weighted distribution:** Send a larger share of traffic to
    higher-capacity servers.
4.  **Health-based routing:** Stop sending traffic to unhealthy servers.

If one server fails, the load balancer can detect the failure and route
requests to healthy servers, assuming enough capacity remains.

**AWS example:** Application Load Balancer (ALB) with target groups and
health checks.

## 8. Stateful vs. stateless application servers

### Stateful server

A stateful server stores information about a user's earlier requests
locally. If the next request reaches another server, that server may not
have the required session information.

### Stateless server

A stateless server does not depend on information stored locally from a
previous request. Any application server can handle the next request.

**Example:** A user's first request reaches Server A, the next reaches
Server B, and the third reaches Server C. All three must be able to
process the requests correctly.

### Where can shared state live?

-   A database for persistent application data.
-   Redis / Amazon ElastiCache for shared session or frequently accessed
    state.
-   Shared object storage such as Amazon S3 for files.
-   A signed token such as JWT for carrying verifiable authentication
    claims.

> **Key principle:** Keep application servers stateless where practical
> so requests can be routed to any healthy server.

## 9. Why statelessness matters for horizontal scaling

Suppose an application has three servers behind a load balancer. A
user's requests may be distributed across all three. If session data
exists only in Server A's memory, a request routed to Server B may fail
to recognize the session.

Moving shared state to an external store, or using an appropriate
token-based approach, prevents each server from depending on its own
local request history.

## 10. Choosing an architecture

The instructor's guiding principle is:

> Choose the simplest architecture that meets projected requirements,
> but no simpler than that.

Avoid over-engineering a small application.

  -----------------------------------------------------------------------
  Application / requirement           Possible starting point
  ----------------------------------- -----------------------------------
  Small personal website              Single server

  Small internal tool                 Single server or vertical scaling

  Growing web application             Separate application and database
                                      tiers

  High-traffic application            Horizontal application scaling

  Large distributed application       Horizontal scaling plus database,
                                      caching, and other distributed
                                      components as needed
  -----------------------------------------------------------------------

For large system-design interview problems, horizontal scaling is often
a reasonable starting point.

## 11. Practical AWS architecture example

### Illustrative request flow

1.  **Route 53** resolves the application's domain name.
2.  An **Application Load Balancer** receives incoming HTTP/HTTPS
    requests.
3.  The ALB forwards requests to healthy application instances.
4.  An **Auto Scaling group** adjusts the number of instances based on
    configured scaling policies.
5.  Application instances access a managed database such as **Amazon
    RDS**.
6.  If shared session state is needed, it can be stored in
    **ElastiCache**.

This is one possible AWS implementation. The correct design depends on
requirements, traffic patterns, availability targets, data access
patterns, and cost.

## 12. System design interview questions

Try answering these without looking at your notes:

1\. **What is scalability?**\
The ability of a system to handle increasing workloads by adding resources without significantly degrading performance.

2\. **What is the difference between vertical and horizontal scaling?**\
Vertical scaling increases resources of an existing server, whereas horizontal scaling adds more servers to distribute the workload.

3\. **Why does vertical scaling have limitations?**\
Because a server has maximum hardware capacity, increasing costs, and potential downtime during upgrades.

4\. **What is a Single Point of Failure (SPOF)?**\
A component whose failure can cause the entire system or a critical part of it to become unavailable.

5\. **Why do we need a load balancer?**\
To distribute incoming traffic across multiple servers, improving availability, scalability, and performance.

6\. **What happens if one application server goes down?**\
The load balancer detects the unhealthy server and routes traffic to the remaining healthy servers.

7\. **What is the difference between stateful and stateless servers?**\
Stateful servers maintain client-specific information locally, whereas stateless servers don't depend on locally stored state between requests.

8\. **Why is stateless architecture important for horizontal scaling?**\
Because any server can handle any request without depending on another server's local session or state.

9\. **Where can session information be stored?**\
In application memory, Redis, a database, or through client-side tokens such as JWT.

10\. **When would you choose vertical scaling over horizontal scaling?**\
When traffic is moderate, a single server has sufficient capacity, or distributing the workload across multiple servers adds unnecessary complexity.

## 13. Quick revision sheet

  Concept              Remember
  -------------------- -------------------------------------------------------------
  Scalability          Ability to handle increasing workload
  Vertical scaling     Bigger machine (scale up)
  Horizontal scaling   More machines (scale out)
  Load balancer        Distributes incoming traffic
  Stateless server     Any healthy server can handle any request
  SPOF                 A single component whose failure can bring down the service

## 14. Topics to study next

The lecture introduces horizontal scaling and notes that database
scaling will be covered separately. Keep these as follow-on topics
rather than mixing them into the current lecture notes:

-   Database read replicas
-   Database sharding
-   Database partitioning
-   Database replication
-   Caching
-   Content Delivery Networks (CDNs)
-   Database connection pooling
-   Auto Scaling policies

## 15. Suggested GitHub notes structure

``` text
system-design/
├── 01-fundamentals/
│   ├── scalability.md
│   ├── availability.md
│   ├── reliability.md
│   └── latency-throughput.md
├── 02-networking/
│   ├── dns.md
│   ├── load-balancers.md
│   └── cdn.md
├── 03-databases/
│   ├── sql-vs-nosql.md
│   ├── replication.md
│   ├── sharding.md
│   └── partitioning.md
├── 04-caching/
│   ├── redis.md
│   └── caching-strategies.md
└── 05-aws-implementation/
    ├── ec2-auto-scaling.md
    ├── application-load-balancer.md
    └── rds-scaling.md
```

## Recommended learning workflow

For each system-design concept, capture three layers:

1.  **Theory:** Understand what the concept means and which problem it
    solves.
2.  **AWS implementation:** Identify the AWS services and understand how
    they implement the concept.
3.  **Interview application:** Explain when and why you would choose the
    concept in a design.

Finish the theoretical concept first. Then map it to AWS. When you solve
a system-design problem, practice deciding where the concept belongs in
the architecture.

------------------------------------------------------------------------

## Source note

Core lecture concepts and framing are based on the supplied Frank Kane
scalability transcript. AWS service mappings and the cricket-community
example are implementation illustrations added to make the notes
practical; they are not claims made by the lecture.
