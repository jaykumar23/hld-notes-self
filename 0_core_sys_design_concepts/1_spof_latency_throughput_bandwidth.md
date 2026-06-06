These two concepts appear in almost every System Design interview because they directly affect how you design scalable systems.

---

# 1. Single Point of Failure (SPOF)

## Definition

A Single Point of Failure (SPOF) is any component whose failure causes the entire system (or a critical part of it) to stop working.

If one thing breaks and the whole system goes down, that's an SPOF.

---

## Simple Example

### Bad Design

```text
Users
   |
Load Balancer
   |
Web Server
   |
Database
```

Suppose:

```text
Database crashes
```

Result:

```text
Entire application stops
```

Database is an SPOF.

---

## Real-World Example

### ATM Network

```text
ATM
  |
Bank Server
```

If the bank server fails:

```text
No ATM can withdraw money
```

Bank server = SPOF.

---

# SPOFs in System Design

## 1. Single Database

```text
App
 |
DB
```

DB failure:

```text
Everything fails
```

---

## 2. Single Load Balancer

```text
Users
  |
Load Balancer
  |
Servers
```

Load balancer crashes:

```text
Nobody reaches servers
```

SPOF.

---

## 3. Single Cache

```text
App
 |
Redis
```

Redis failure may overload DB.

Potential SPOF.

---

## 4. Single Service

```text
Order Service
```

If all orders depend on it:

```text
Order Service down
→ Entire ordering system down
```

---

# Removing SPOFs

## Database Replication

Instead of:

```text
App
 |
DB
```

Use:

```text
        Primary
           |
      -------------
      |           |
 Replica1     Replica2
```

If primary fails:

```text
Replica becomes primary
```

No downtime.

---

## Multiple Load Balancers

Instead of:

```text
Users
  |
LB
```

Use:

```text
          DNS
           |
    ----------------
    |              |
   LB1           LB2
```

If LB1 fails:

```text
Traffic → LB2
```

---

## Multiple App Servers

```text
      LB
       |
-------------------
|        |        |
S1       S2       S3
```

One server dies:

```text
Others serve traffic
```

---

## Multi-Region Deployment

```text
Mumbai
Delhi
Bangalore
```

If Mumbai data center fails:

```text
Traffic → Delhi
```

---

# Interview Example

Suppose interviewer asks:

> Design an Online Shopping System.

Bad answer:

```text
1 Load Balancer
1 App Server
1 Database
```

Interviewer immediately sees:

```text
3 SPOFs
```

Better:

```text
2 Load Balancers
Multiple App Servers
Primary + Replica DBs
```

---

# SPOF Interview Answer

> A Single Point of Failure is any component whose failure can bring down the entire system. In HLD we eliminate SPOFs using redundancy, replication, failover mechanisms, and deploying multiple instances of critical services.

---

# 2. Latency vs Throughput vs Bandwidth

This is one of the most common interview topics.

Many candidates mix them up.

---

# Latency

## Definition

Latency is the time taken for one request to travel through the system and receive a response.

Measured in:

```text
Milliseconds (ms)
Microseconds (μs)
```

---

## Example

User clicks:

```text
Open Amazon Product Page
```

Response arrives after:

```text
100 ms
```

Latency:

```text
100 ms
```

---

## Highway Example

Imagine cars on a road.

Latency =

```text
Time taken by one car
to reach destination
```

---

## Lower Latency = Faster Response

Good:

```text
50 ms
```

Bad:

```text
1000 ms
```

---

# Throughput

## Definition

Throughput is the amount of work completed per unit time.

Measured as:

```text
Requests/second
Transactions/second
Messages/second
```

---

## Example

Server handles:

```text
10,000 requests/sec
```

Throughput:

```text
10,000 RPS
```

---

## Highway Example

Throughput =

```text
Number of cars
passing every second
```

---

## High Throughput

Good for:

* YouTube
* Instagram
* Amazon
* Payment Systems

---

# Bandwidth

## Definition

Bandwidth is the maximum amount of data that can be transferred per second.

Measured in:

```text
Mbps
Gbps
TB/s
```

---

## Example

Internet connection:

```text
1 Gbps
```

Bandwidth:

```text
1 Gigabit per second
```

---

## Highway Example

Bandwidth =

```text
Number of lanes
```

More lanes:

```text
More data can travel
```

---

# Visual Comparison

Imagine a water pipe.

### Bandwidth

```text
Pipe Width
```

How much water can fit.

---

### Latency

```text
Time for water
to reach the other end
```

---

### Throughput

```text
Actual water delivered
per second
```

---

# Example

Suppose:

```text
Bandwidth = 1 Gbps
Latency = 50 ms
```

Server processes:

```text
5000 requests/sec
```

Then:

```text
Bandwidth = Capacity
Latency = Delay
Throughput = Actual Work Done
```

---

# Relationship

## High Bandwidth ≠ Low Latency

Example:

```text
100 Gbps network
```

But data center is far away.

Latency:

```text
300 ms
```

Bandwidth high.

Latency high too.

---

## Low Latency ≠ High Throughput

Example:

```text
Server responds in 1 ms
```

But can handle:

```text
10 requests/sec
```

Low latency.

Poor throughput.

---

## High Throughput Usually Requires

* Horizontal scaling
* Load balancing
* Caching
* Efficient DB queries
* Message queues

---

# Interview Example: YouTube

### Latency

Video should start quickly.

```text
< 200 ms
```

---

### Throughput

Millions of viewers simultaneously.

```text
Millions of requests/sec
```

---

### Bandwidth

Huge video transfer.

```text
Many Gbps/Tbps
```

Needed across CDNs.

---

# Common Interview Question

### Which is more important?

Answer:

Depends on the system.

### Stock Trading

Prioritize:

```text
Low Latency
```

Milliseconds matter.

---

### Video Streaming

Prioritize:

```text
High Bandwidth
```

Large files.

---

### Social Media

Prioritize:

```text
High Throughput
```

Millions of requests.

---

# Quick Revision Table

| Concept    | Meaning                        | Unit      |
| ---------- | ------------------------------ | --------- |
| Latency    | Time for one request           | ms        |
| Throughput | Requests processed per second  | RPS/TPS   |
| Bandwidth  | Maximum data transfer capacity | Mbps/Gbps |

### Easy Memory Trick

**Latency = Delay**

```text
"How long?"
```

**Throughput = Work Done**

```text
"How many requests?"
```

**Bandwidth = Capacity**

```text
"How much data can fit?"
```

---

### Interview One-Liner

> Latency measures response time for a request, throughput measures how many requests a system can process per unit time, and bandwidth measures the maximum amount of data that can be transferred through the network per unit time. High-performance systems aim to balance all three while avoiding Single Points of Failure through redundancy and failover mechanisms.
