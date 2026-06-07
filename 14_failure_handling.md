These are very important topics in **High-Level Design (HLD)** interviews because distributed systems assume that **machines, networks, and services can fail at any time**.

---

# 1. Heartbeats

## What is a Heartbeat?

A heartbeat is a periodic signal sent by a server/service to indicate:

> "I'm alive and working."

Just like checking a person's pulse.

---

## Why Do We Need Heartbeats?

Imagine:

```text
Load Balancer
      |
 -----------------
 |       |       |
S1      S2      S3
```

If S2 crashes, the load balancer needs a way to know that.

Heartbeats solve this problem.

---

## How It Works

Every few seconds:

```text
S1 → Alive
S2 → Alive
S3 → Alive
```

The monitoring service keeps receiving heartbeats.

---

### Normal Scenario

```text
Time 0   → heartbeat
Time 5   → heartbeat
Time 10  → heartbeat
Time 15  → heartbeat
```

Server is healthy.

---

### Failure Scenario

```text
Time 0   → heartbeat
Time 5   → heartbeat
Time 10  → heartbeat
Time 15  → nothing
Time 20  → nothing
```

System assumes:

```text
Server may be dead
```

---

## Heartbeat Architecture

```text
            Heartbeats
Server A -----------------> Monitor

Server B -----------------> Monitor

Server C -----------------> Monitor
```

Monitor tracks all nodes.

---

## Types of Heartbeats

### Push Model

Server sends heartbeat.

```text
Server → Monitor
```

Most common.

Example:

* Kubernetes Nodes
* ZooKeeper
* Eureka

---

### Pull Model

Monitor asks server.

```text
Monitor → Ping
Server → Alive
```

Used in health checks.

Example:

* Load balancers
* Monitoring tools

---

## Heartbeat Interval Tradeoff

### Frequent Heartbeats

```text
every 1 second
```

Pros:

* Detect failures quickly

Cons:

* More network traffic

---

### Infrequent Heartbeats

```text
every 30 seconds
```

Pros:

* Less overhead

Cons:

* Slow failure detection

---

### Interview Answer

Typical systems use:

```text
Heartbeat every 5-10 seconds
```

with

```text
Miss 3 heartbeats => mark node unhealthy
```

---

# 2. Handling Failures in Distributed Systems

---

## Biggest Reality

In distributed systems:

> Failures are not exceptions. They are expected.

Servers crash.
Networks fail.
Databases become unavailable.

Design assuming failure.

---

## Types of Failures

### 1. Server Failure

```text
Server crashes
```

Example:

* Machine shutdown
* Process killed

---

### 2. Network Failure

```text
Server alive
Network dead
```

Most common.

```text
A ❌ B
```

Both may think the other is dead.

---

### 3. Disk Failure

```text
Data lost
```

Hardware corruption.

---

### 4. Database Failure

```text
DB unavailable
```

Application cannot serve requests.

---

### 5. Dependency Failure

```text
Payment Service Down
```

Your service may still be running.

---

# Common Failure Handling Strategies

---

## Strategy 1: Replication

Keep copies.

```text
Primary
   |
-----------
|         |
Replica1 Replica2
```

If primary fails:

```text
Replica becomes primary
```

Examples:

* MySQL Replication
* MongoDB Replica Set

---

## Strategy 2: Retry

Temporary failures happen.

```text
Request Failed
      |
    Retry
```

Example:

```text
Network timeout
```

Retry after:

```text
100ms
500ms
1000ms
```

---

## Exponential Backoff

Instead of:

```text
retry immediately
```

Use:

```text
1s
2s
4s
8s
```

Prevents overload.

---

## Strategy 3: Failover

Switch to backup service.

```text
Primary Server
      ↓
Failure
      ↓
Backup Server
```

Used everywhere.

Examples:

* Databases
* Kafka Brokers
* Redis

---

## Strategy 4: Circuit Breaker

Prevents repeated failures.

Without breaker:

```text
App → Dead Service
App → Dead Service
App → Dead Service
```

Thousands of useless requests.

---

### With Circuit Breaker

```text
Too many failures
      ↓
Circuit Opens
      ↓
Stop sending requests
```

Service gets time to recover.

---

## Strategy 5: Graceful Degradation

Show limited functionality.

Example:

```text
Netflix Recommendation Service Down
```

Still play videos.

Don't fail entire system.

---

# Interview Example

### E-commerce Site

Payment service down.

Bad:

```text
Entire website crashes
```

Good:

```text
Browsing works
Cart works
Payment disabled temporarily
```

---

# 3. Failure Detection Mechanisms

---

## Problem

How do we know a machine failed?

This is harder than it sounds.

---

### Scenario

Node A sends request.

```text
A → B
```

No response.

Question:

```text
Did B crash?
OR
Network failed?
OR
Response delayed?
```

Impossible to know with certainty.

This is a famous distributed systems problem.

---

## Mechanism 1: Timeout

Most common.

```text
Request
```

Wait:

```text
5 seconds
```

No response:

```text
Failure suspected
```

---

### Example

```text
API Call
Timeout = 2 sec
```

If no reply:

```text
Mark failed
```

---

## Mechanism 2: Heartbeats

Already discussed.

```text
Miss N heartbeats
```

Node considered dead.

---

## Mechanism 3: Gossip Protocol

Nodes exchange health information.

```text
A knows B
B knows C
C knows D
```

Information spreads like rumors.

---

### Example

```text
A says B failed
```

Soon:

```text
C knows
D knows
E knows
```

Used in:

* Cassandra
* DynamoDB-inspired systems
* Consul

---

## Mechanism 4: Health Checks

Load balancer continuously checks servers.

```text
GET /health
```

Response:

```json
{
  "status": "healthy"
}
```

If unhealthy:

```text
Remove from traffic
```

---

### Architecture

```text
Load Balancer
      |
  Health Check
      |
   Server
```

---

## Mechanism 5: Leader Election

Cluster must know if leader died.

```text
Leader
  |
Followers
```

Leader crashes.

Followers detect failure and elect new leader.

Used in:

* ZooKeeper
* etcd
* Kubernetes

---

# False Positives

Very important interview topic.

---

## What is a False Positive?

System thinks:

```text
Node Dead
```

But node is actually alive.

Reason:

```text
Network delay
```

Example:

```text
Heartbeat delayed
```

instead of

```text
Node crashed
```

---

## Why Dangerous?

May trigger:

```text
Failover
Replication
Leader Election
```

when not needed.

---

## Solution

Don't mark dead immediately.

Use:

```text
Miss 3 heartbeats
```

instead of

```text
Miss 1 heartbeat
```

---

# Real HLD Example: WhatsApp Server Failure

```text
Users
  |
Load Balancer
  |
--------------------
|        |         |
S1       S2        S3
```

Every server sends heartbeat.

```text
Heartbeat → Monitoring Service
```

S2 crashes.

```text
No heartbeat
```

Monitoring system detects:

```text
S2 unhealthy
```

Load balancer removes S2.

Traffic routed to:

```text
S1 and S3
```

Users continue chatting.

This is the essence of **fault-tolerant distributed systems**.

---

# 5-Minute HLD Interview Revision

### Heartbeats

* Periodic "I'm alive" signal
* Detect failed nodes
* Push or Pull model
* Missing multiple heartbeats ⇒ suspected failure

### Handling Failures

* Replication
* Retry + Exponential Backoff
* Failover
* Circuit Breaker
* Graceful Degradation

### Failure Detection

* Timeouts
* Heartbeats
* Health Checks
* Gossip Protocol
* Leader Election

### Important Interview Line

> In distributed systems, you can never know with 100% certainty that a node has failed. You can only suspect failure based on missing heartbeats, timeouts, or health checks. Therefore systems are designed to tolerate failures rather than completely avoid them.

This statement often impresses interviewers because it shows understanding of real-world distributed system behavior.
