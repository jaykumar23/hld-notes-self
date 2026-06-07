These topics are extremely important because **many candidates know the components (Redis, Kafka, Load Balancer, DB Sharding) but fail the interview due to poor design approach.**

Think of HLD interviews as:

> **Can you think like a senior engineer and design a system step-by-step?**

---

# 1. Requirement Clarification and Assumptions

## Why Interviewers Care

Most candidates immediately start drawing boxes.

Senior engineers first understand the problem.

Example:

Design YouTube.

Bad candidate:

> We'll use CDN, Kafka, Cassandra...

Good candidate:

> Let's first understand requirements.

---

## Functional Requirements

What system should do.

Example:

For YouTube:

* Upload videos
* Watch videos
* Search videos
* Like/comment
* Recommendations

---

## Non-Functional Requirements

How system should behave.

Example:

* High availability
* Low latency
* Scalability
* Durability
* Reliability

---

## Ask Clarifying Questions

Example:

Design WhatsApp.

Questions:

* 1-to-1 chat only?
* Group chat?
* Voice/video?
* Message history?
* End-to-end encryption?

---

## State Assumptions

Interviewers love this.

Example:

> Assume 100M DAU.
>
> Assume 80% reads, 20% writes.
>
> Assume messages stored forever.

Now everyone is working with same assumptions.

---

# Interview Framework

Always start with:

```
1. Clarify requirements
2. Define assumptions
3. Estimate scale
4. Design APIs
5. High-level architecture
6. Database design
7. Scaling
8. Bottlenecks
9. Tradeoffs
```

This alone makes you look organized.

---

# 2. Capacity Estimation

This separates mid-level from senior candidates.

---

## Example

Design Instagram.

Assume:

* 100M DAU
* 10M photos/day

Photo size:

```
2 MB
```

Daily storage:

```
10M × 2MB
= 20 TB/day
```

Yearly:

```
20 × 365
= 7300 TB
≈ 7.3 PB
```

Now interviewer knows:

> Candidate understands scale.

---

## Traffic Estimation

Assume:

```
100M users
```

Daily requests:

```
100 requests/user/day
```

Total:

```
10 Billion requests/day
```

QPS:

```
10B / 86400

≈115K requests/sec
```

Peak:

```
2-5x average

≈500K QPS
```

Design for peak.

---

## Network Bandwidth

Example:

500K requests/sec

Each response:

```
100 KB
```

Bandwidth:

```
500K × 100 KB

= 50 GB/sec
```

Need CDN.

---

## Database Capacity

Estimate:

* records
* growth rate
* retention period

Interviewers care more about methodology than exact numbers.

---

# 3. Bottleneck Identification

This is where most interviews become interesting.

After architecture:

Ask yourself:

> What breaks first?

---

## Database

Most common bottleneck.

Example:

Single MySQL instance.

Problems:

* CPU
* memory
* connections
* storage

Solution:

* replicas
* sharding

---

## Cache

Problem:

Cache miss storm.

Example:

Popular celebrity posts.

Millions hit DB simultaneously.

Solution:

* cache warming
* request coalescing

---

## Load Balancer

Can become bottleneck.

Solution:

```
LB → LB → Servers
```

or multiple LBs.

---

## Kafka

Problem:

Too few partitions.

Consumers can't scale.

Solution:

Increase partitions.

---

## Network

Large media systems:

* Netflix
* YouTube

Network becomes bottleneck.

Solution:

CDN.

---

## Single Point of Failure

Interviewers love asking:

> What if this node dies?

Always identify SPOFs.

---

# 4. Tradeoff Justification

The MOST important skill.

There is no perfect design.

Every design is a compromise.

---

## SQL vs NoSQL

SQL

Pros:

* ACID
* joins
* consistency

Cons:

* scaling harder

NoSQL

Pros:

* horizontal scaling
* flexibility

Cons:

* eventual consistency

---

## Cache vs Freshness

Aggressive cache:

Pros:

* fast

Cons:

* stale data

Example:

Instagram likes count may be delayed.

Acceptable.

Bank balance?

Not acceptable.

---

## Strong vs Eventual Consistency

Strong:

```
Bank
Payments
Ticket booking
```

Eventual:

```
Social media
Analytics
Recommendations
```

---

## Availability vs Consistency

From CAP theorem.

If network partition occurs:

Choose:

* Consistency
* Availability

Can't have both.

---

## Monolith vs Microservices

Monolith:

Pros:

* simple

Cons:

* scaling difficult

Microservices:

Pros:

* independent scaling

Cons:

* operational complexity

---

# Interview Trick

Whenever you choose something say:

> I am choosing X because Y.
>
> Tradeoff is Z.

Example:

> I am choosing Cassandra because write throughput is critical.
>
> Tradeoff is weaker consistency.

Interviewers love hearing this.

---

# 5. Common System Design Mistakes

---

## Mistake #1

Jumping into architecture.

Bad:

```
User
→ LB
→ Redis
→ Kafka
→ Cassandra
```

without requirements.

---

## Mistake #2

No scale estimation.

Everything looks easy until:

```
100 million users
```

---

## Mistake #3

Using every buzzword.

Bad:

```
Kafka
Redis
ElasticSearch
Spark
Flink
Cassandra
Kubernetes
```

Why?

"I saw it on YouTube."

---

## Mistake #4

Ignoring failures.

Always answer:

```
What if service crashes?
What if DB dies?
What if cache dies?
```

---

## Mistake #5

No bottleneck discussion.

Senior engineers proactively discuss scaling issues.

---

## Mistake #6

No tradeoffs.

Every decision should have:

```
Reason
Benefits
Drawbacks
```

---

## Mistake #7

Over-engineering.

Designing:

```
Netflix-scale architecture
```

for

```
10,000 users
```

---

# 6. End-to-End Case Study

Let's design a URL Shortener.

Like:

```
bit.ly/abc123
```

---

## Step 1

Requirements

Functional:

* Create short URL
* Redirect

Non-functional:

* High availability
* Low latency

---

## Step 2

Estimate

100M URLs

Average URL:

```
500 bytes
```

Storage:

```
50 GB
```

Easy.

---

## Step 3

API

Create:

```http
POST /shorten
```

Redirect:

```http
GET /abc123
```

---

## Step 4

Architecture

```text
Users
   |
Load Balancer
   |
App Servers
   |
Database
```

---

## Step 5

Short Code Generation

Use:

```text
Base62

a-z
A-Z
0-9
```

Example:

```
12345
→ dnh
```

---

## Step 6

Caching

Popular links:

```text
Redis
```

Flow:

```text
Redis
  ↓ miss
DB
  ↓
Redis
```

---

## Step 7

Scaling

Read-heavy system.

Add:

```text
CDN
Redis
Read Replicas
```

---

## Step 8

Bottlenecks

### DB Hotspots

Popular URL.

Solution:

* Cache

### Cache Failure

Fallback:

* Database

### DB Failure

Solution:

* Replicas

---

## Step 9

Tradeoffs

Use Redis:

Pros:

* Fast

Cons:

* Extra cost
* Cache invalidation

---

# Golden HLD Interview Template

Memorize this flow:

```text
1. Clarify requirements
2. Define assumptions
3. Estimate scale
4. APIs
5. High-level architecture
6. Database schema
7. Scaling strategy
8. Bottlenecks
9. Failures
10. Tradeoffs
11. Future improvements
```

For almost every HLD interview question (Uber, WhatsApp, YouTube, Netflix, BookMyShow, Google Docs, Instagram, TinyURL), following this sequence will keep the discussion structured and help you cover the points interviewers expect from senior engineers.
