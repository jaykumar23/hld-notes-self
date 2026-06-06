These are the **3 most frequently asked HLD interview concepts**. Interviewers expect you to understand the trade-offs, not just definitions.

---

# 1. Scalability

### Definition

Scalability is the ability of a system to handle increasing traffic, users, or data without significant performance degradation.

### Example

Suppose your application handles:

```text
10,000 requests/day
```

After a year:

```text
10 million requests/day
```

Can your system still work efficiently?

If yes → scalable.

---

## Types of Scaling

### Vertical Scaling (Scale Up)

Increase resources of the same machine.

```text
4 CPU → 16 CPU
16 GB RAM → 64 GB RAM
```

Example:

```text
Server A
↓
More CPU/RAM
```

### Pros

* Simple
* No code changes

### Cons

* Hardware limit exists
* Expensive
* Single Point of Failure

---

### Horizontal Scaling (Scale Out)

Add more servers.

```text
1 server → 10 servers → 100 servers
```

Example:

```text
         Load Balancer
               |
    -----------------------
    |         |          |
 Server1  Server2   Server3
```

### Pros

* Practically unlimited growth
* Better fault tolerance
* Cheaper than huge machines

### Cons

* Distributed system complexity
* Data synchronization challenges

---

## Interview Example

### YouTube

When users increase:

```text
Add more web servers
Add more cache servers
Add more DB replicas
```

Instead of buying one giant server.

---

## HLD Answer

When interviewer asks:

> How would you scale your system?

Mention:

* Load Balancer
* Stateless Services
* Caching
* Database Replication
* Sharding
* CDN
* Queue-based processing

---

# 2. Availability

### Definition

Availability measures how often a system is operational and accessible.

Formula:

```text
Availability =
Uptime / (Uptime + Downtime)
```

---

## Example

Website works:

```text
364 days
```

Down:

```text
1 day
```

Availability:

```text
364 / 365 = 99.73%
```

---

## Availability Percentages

### 99%

```text
~3.65 days downtime/year
```

### 99.9%

```text
~8.7 hours/year
```

### 99.99%

```text
~52 minutes/year
```

### 99.999% (Five Nines)

```text
~5 minutes/year
```

Used by:

* Banking
* Payment systems
* Cloud providers

---

## How To Improve Availability

### Redundancy

Don't rely on one server.

Bad:

```text
User
 |
Server
```

Server crash = website down.

---

Good:

```text
      Load Balancer
           |
    ----------------
    |              |
 Server1      Server2
```

One fails → other serves traffic.

---

### Multi-AZ Deployment

Deploy in multiple data centers.

```text
Mumbai
Delhi
Bangalore
```

If one data center fails, others continue.

---

### Replication

```text
Primary DB
    |
Replicas
```

If one DB fails:

```text
Switch to replica
```

---

## Interview Example

### Netflix

If one service crashes:

```text
Traffic routed elsewhere
```

Users continue watching movies.

High Availability.

---

# 3. Reliability

### Definition

Reliability means the system consistently performs correctly and returns accurate results.

---

## Availability vs Reliability

Many candidates confuse these.

### Available but Not Reliable

Example:

```text
Bank website opens
```

But:

```text
Balance = Wrong
Transactions fail
```

Website is available.

But not reliable.

---

### Reliable but Not Available

Example:

```text
Database gives correct data
```

But:

```text
Offline for 5 hours
```

Reliable when running.

Not available.

---

## Reliability Questions

Can the system:

* Avoid data loss?
* Process requests correctly?
* Recover after failures?
* Maintain data consistency?

---

## How To Improve Reliability

### Replication

Store data in multiple places.

```text
Primary
   |
Replica1
Replica2
```

---

### Backups

```text
Daily Backup
```

If database is corrupted:

```text
Restore backup
```

---

### Retry Mechanisms

Network failure:

```text
Payment Request
    ↓
Fail
    ↓
Retry
```

---

### Message Queues

Instead of losing requests:

```text
Producer
   |
 Queue
   |
Consumer
```

Requests remain safe.

---

### Idempotency

Prevent duplicate operations.

Example:

```text
Pay ₹1000
```

Network timeout.

User retries.

Without idempotency:

```text
₹1000 deducted twice
```

With idempotency:

```text
Only one payment processed
```

---

# Real-World Example: Amazon Checkout

## Scalability

During sale:

```text
1 million → 50 million users
```

Add more servers.

---

## Availability

If one server crashes:

```text
Traffic → Other servers
```

Users continue shopping.

---

## Reliability

After payment:

```text
Order must be saved correctly
```

No duplicate orders.

No lost transactions.

---

# Quick Interview Comparison

| Concept      | Meaning           | Goal               |
| ------------ | ----------------- | ------------------ |
| Scalability  | Handle growth     | More users/traffic |
| Availability | System is up      | Less downtime      |
| Reliability  | Correct operation | Accurate results   |

---

# Interview One-Liner

**Scalability**

> Ability of a system to handle increasing load by adding resources.

**Availability**

> Percentage of time the system remains operational and accessible.

**Reliability**

> Ability of a system to consistently perform correctly without losing data or producing incorrect results.

A strong HLD answer often sounds like:

> "The system should be horizontally scalable to handle traffic growth, highly available through redundancy and failover mechanisms, and reliable by ensuring data durability, replication, retries, and fault-tolerant design."
