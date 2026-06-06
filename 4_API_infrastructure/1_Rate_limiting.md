# Rate Limiting (HLD Interview Guide)

Rate Limiting is one of the most commonly discussed topics in System Design interviews.

It protects systems from:

* Abuse
* DDoS attacks
* Bots
* Excessive traffic
* Cost explosions

---

# What is Rate Limiting?

Rate Limiting controls **how many requests a user/client can make within a specific time period.**

Example:

```text
100 requests/minute
```

If a user sends:

```text
Request #101
```

Server rejects it.

Response:

```http
429 Too Many Requests
```

---

# Real World Example

Imagine a movie theater.

The theater allows:

```text
100 people/hour
```

If 101st person arrives:

```text
Sorry, come back later.
```

Rate limiting works similarly.

---

# Why Do We Need Rate Limiting?

Without rate limiting:

```text
Attacker
    |
100,000 req/sec
    |
Server
```

Problems:

### Resource Exhaustion

CPU becomes overloaded.

```text
CPU = 100%
```

---

### Database Overload

```text
Millions of queries
```

Database crashes.

---

### DDoS Protection

Attackers flood APIs.

Rate limiting helps reduce impact.

---

### Fair Usage

One user shouldn't consume all resources.

Example:

```text
User A = 1 request
User B = 10000 requests
```

Without limits:

```text
User B monopolizes system
```

---

### Cost Control

Cloud services charge for:

* API requests
* Database reads
* Network traffic

Rate limiting reduces costs.

---

# Where Is Rate Limiting Applied?

Usually at:

### API Gateway

```text
Client
   |
API Gateway
   |
Services
```

Most common.

---

### Load Balancer

```text
Client
   |
Load Balancer
   |
Servers
```

---

### Reverse Proxy

```text
NGINX
Envoy
HAProxy
```

---

### Application Layer

```text
Inside Service Code
```

Less common for large systems.

---

# Types of Rate Limiting

---

# 1. Per User

```text
100 requests/minute per user
```

Example:

```text
User123 → 100/min
User456 → 100/min
```

Most common.

---

# 2. Per IP

```text
100 requests/minute/IP
```

Useful for:

* Anonymous traffic
* Public APIs

---

# 3. Per API Key

```text
1000 requests/day
```

Used in SaaS products.

---

# 4. Per Organization

```text
Company A → 10,000/hour
Company B → 100,000/hour
```

Enterprise APIs.

---

# 5. Per Endpoint

Example:

```text
Login API → 5/min
Search API → 1000/min
```

Different endpoints get different limits.

---

# Hard Limit vs Soft Limit

### Hard Limit

After limit reached:

```text
Request blocked
```

Example:

```text
100 requests/min
101st blocked
```

---

### Soft Limit

Allow temporary bursts.

Example:

```text
100 limit
Allowed burst = 20
```

User can briefly exceed limit.

---

# Popular Rate Limiting Algorithms

Interviewers love this section.

---

# 1. Fixed Window Counter

Simplest algorithm.

---

### Example

Limit:

```text
100 requests/minute
```

Counter resets every minute.

```text
10:00 -> counter = 0
10:00-10:01 -> count requests
10:01 -> reset
```

---

### Example

```text
Limit = 5/min
```

User sends:

```text
10:00:55 -> 5 requests
```

Allowed.

Then:

```text
10:01:01 -> 5 more requests
```

Allowed again.

Total:

```text
10 requests in 6 seconds
```

Problem:

```text
Traffic spikes at boundaries
```

---

### Complexity

```text
Space: O(users)
Time: O(1)
```

Simple but inaccurate.

---

# 2. Sliding Window Log

Stores timestamp of every request.

---

Example:

```text
Limit = 5/min
```

Store:

```text
10:00:01
10:00:10
10:00:20
10:00:30
10:00:40
```

When new request arrives:

Remove timestamps older than:

```text
CurrentTime - 60 sec
```

Count remaining.

---

### Advantage

Very accurate.

---

### Disadvantage

Memory heavy.

```text
Millions of users
Millions of timestamps
```

---

### Complexity

```text
Time: O(log n)
Space: O(requests)
```

---

# 3. Sliding Window Counter

Hybrid approach.

Most frequently discussed in interviews.

---

Instead of storing every request:

Store counters for windows.

Example:

```text
Previous Window = 80 requests
Current Window = 20 requests
```

Calculate weighted estimate.

Provides:

```text
Better accuracy
Less memory
```

Used in many production systems.

---

# 4. Token Bucket (Very Important)

Most popular interview algorithm.

---

Imagine a bucket.

```text
Capacity = 10 tokens
```

Tokens refill continuously.

```text
1 token/sec
```

Every request consumes:

```text
1 token
```

---

### Example

```text
Bucket size = 10
Refill = 1/sec
```

Initially:

```text
10 tokens
```

User sends:

```text
5 requests
```

Remaining:

```text
5 tokens
```

After 5 seconds:

```text
Bucket refilled to 10
```

---

### Benefits

Allows bursts.

```text
10 requests instantly
```

Then slows down.

Perfect for APIs.

---

### Used By

* AWS API Gateway
* Networking devices
* Many cloud providers

---

# 5. Leaky Bucket

Think of a bucket leaking water.

Incoming requests:

```text
Fast
```

Outgoing processing:

```text
Constant rate
```

---

Example

```text
Incoming:
100 requests instantly
```

Output:

```text
10 requests/sec
```

Smooth traffic.

---

### Benefit

Prevents sudden spikes.

---

### Drawback

Can increase latency.

---

# Token Bucket vs Leaky Bucket

### Token Bucket

Allows bursts.

```text
✓ Burst Friendly
```

Example:

```text
10 requests instantly
```

Allowed.

---

### Leaky Bucket

Smooth traffic.

```text
✓ Constant Output Rate
```

No sudden bursts.

---

# Distributed Rate Limiting

This is where interviews become interesting.

---

# Problem

Single server:

```text
Server A
```

Easy.

Counter stored locally.

---

But what if:

```text
LB
 |
 + Server1
 + Server2
 + Server3
```

User sends:

```text
50 req -> Server1
50 req -> Server2
50 req -> Server3
```

Each server thinks:

```text
Only 50 requests
```

Actual:

```text
150 requests
```

Limit bypassed.

---

# Solution: Shared Store

Use Redis.

```text
Client
   |
Gateway
   |
Redis Counter
```

Every server updates same counter.

---

# Why Redis?

Redis offers:

```text
INCR
EXPIRE
```

Atomic operations.

Fast.

In-memory.

---

# Redis Example

User limit:

```text
100/min
```

Redis key:

```text
user:123
```

Request arrives:

```text
INCR user:123
```

Result:

```text
98
99
100
101
```

If:

```text
count > 100
```

Reject request.

---

# Distributed Challenges

---

## Race Conditions

Multiple servers increment simultaneously.

Need:

```text
Atomic operations
```

Redis solves this.

---

## Redis Failure

What happens if Redis crashes?

Options:

### Fail Open

Allow requests.

```text
Availability ↑
Security ↓
```

---

### Fail Closed

Block requests.

```text
Security ↑
Availability ↓
```

Tradeoff depends on system.

---

# What Metrics Are Usually Limited?

### Login API

```text
5 attempts/min
```

Prevents brute force attacks.

---

### OTP API

```text
3 OTPs/hour
```

Prevents SMS abuse.

---

### Payment API

```text
10 transactions/min
```

Fraud protection.

---

### Search API

```text
1000/min
```

Protects search infrastructure.

---

# How APIs Communicate Rate Limits

Response headers:

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 12
X-RateLimit-Reset: 1719000000
```

Client knows remaining quota.

---

When exceeded:

```http
HTTP/1.1 429 Too Many Requests

Retry-After: 60
```

Meaning:

```text
Try again in 60 seconds
```

---

# HLD Interview Answer

When designing a large-scale API:

```text
Client
   |
API Gateway
   |
Redis (Rate Limit Store)
   |
Microservices
```

Flow:

1. Request reaches API Gateway
2. Gateway checks Redis counter
3. If limit not exceeded → allow
4. If exceeded → return 429
5. Counter automatically expires after window

---

# Interview One-Liner

**Rate Limiting is a mechanism that controls how many requests a client can make within a time window to protect services from abuse, ensure fair usage, improve availability, and control infrastructure costs.**

---

# Quick Revision (30 Seconds)

```text
Why?
✓ Prevent abuse
✓ DDoS protection
✓ Fair usage
✓ Cost control

Where?
✓ API Gateway
✓ Load Balancer
✓ Reverse Proxy

Algorithms:
✓ Fixed Window
✓ Sliding Window Log
✓ Sliding Window Counter
✓ Token Bucket (Most Popular)
✓ Leaky Bucket

Distributed Solution:
✓ Redis
✓ Atomic INCR
✓ TTL/EXPIRE

Response:
✓ HTTP 429 Too Many Requests
✓ Retry-After Header
```

### Interview Tip

If asked:

**"How would you design rate limiting for millions of users?"**

Answer:

> "I would implement rate limiting at the API Gateway layer using a Token Bucket algorithm and store counters/tokens in Redis because Redis provides atomic operations, low latency, horizontal scalability, and centralized state across multiple gateway instances."
