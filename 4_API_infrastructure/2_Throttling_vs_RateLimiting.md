# What is Throttling?

**Throttling** means intentionally **slowing down or restricting traffic** when usage becomes too high.

Instead of completely blocking requests, the system controls the speed at which requests are processed.

---

## Real-Life Example

Imagine a highway toll booth.

### Rate Limiting

```text
Maximum 100 cars per hour
```

101st car is denied entry.

```text
Car 101 → STOP
```

---

### Throttling

```text
All cars allowed
But only 1 car every 5 seconds
```

Cars wait in a queue.

```text
Car 1 → Go
Car 2 → Wait
Car 3 → Wait
```

Traffic is slowed down rather than blocked.

---

# Throttling vs Rate Limiting

Many people use these terms interchangeably, but they are slightly different.

| Feature         | Rate Limiting            | Throttling                 |
| --------------- | ------------------------ | -------------------------- |
| Goal            | Limit number of requests | Control processing speed   |
| Excess Requests | Rejected                 | Delayed or slowed          |
| Response        | Usually HTTP 429         | Often queued/slowed        |
| User Experience | Hard stop                | Slower service             |
| Example         | 100 requests/min         | 10 requests/sec processing |

---

# Example

Suppose:

```text
Limit = 100 requests/min
```

User sends:

```text
150 requests
```

---

## Rate Limiting

```text
First 100 → Allowed
Next 50 → Rejected
```

Response:

```http
429 Too Many Requests
```

---

## Throttling

```text
All 150 accepted
```

But:

```text
Processed slowly
```

Example:

```text
10 requests/sec
```

Some requests wait in queue.

---

# Why Throttling is Useful

---

## Prevent Server Overload

Without throttling:

```text
10000 requests arrive instantly
```

CPU spikes.

Database crashes.

---

With throttling:

```text
10000 requests arrive
↓
Queue
↓
Processed at safe speed
```

System remains stable.

---

## Fair Resource Allocation

Example:

```text
User A → 10 requests
User B → 10000 requests
```

Throttling prevents User B from consuming all resources.

---

## Cost Protection

Cloud services often throttle expensive operations.

Example:

```text
AI Inference
Video Processing
Image Rendering
```

to avoid resource exhaustion.

---

# Types of Throttling

---

## 1. Request Throttling

Limit processing rate.

Example:

```text
100 req/sec
```

Anything beyond waits.

---

## 2. Bandwidth Throttling

Limit network speed.

Example:

```text
10 MB/sec
```

Common in:

* ISPs
* Streaming platforms

---

## 3. CPU Throttling

Limit CPU usage.

Example:

```text
Container A
Maximum CPU = 2 cores
```

Used in Kubernetes and cloud environments.

---

## 4. API Throttling

Cloud providers commonly throttle APIs.

Example:

```text
AWS
Google Cloud
Azure
```

Requests beyond capacity are delayed or limited.

---

# How Throttling is Implemented

---

## Queue-Based Throttling

Most common.

```text
Client
   |
Queue
   |
Workers
```

Requests enter queue.

Workers process at fixed speed.

---

Example:

```text
1000 requests arrive
Workers handle 100/sec
```

Remaining requests wait.

---

## Token Bucket Throttling

```text
Bucket = 100 tokens
Refill = 10/sec
```

Requests consume tokens.

If no tokens remain:

```text
Request waits
```

or gets delayed.

---

# Example in System Design

### Design a URL Shortener

Traffic:

```text
50,000 req/sec
```

Database supports:

```text
10,000 writes/sec
```

Problem:

```text
DB overload
```

Solution:

```text
API Gateway
   |
Queue
   |
Workers
   |
Database
```

Workers throttle write speed.

Database remains healthy.

---

# Where Interviewers Expect Throttling

### API Gateway

```text
Client
   |
API Gateway
```

Common place.

---

### Message Queues

```text
Kafka
RabbitMQ
SQS
```

Consumers process messages at controlled speed.

---

### Databases

Limit expensive queries.

---

### Microservices

Protect downstream services.

---

# Rate Limiting + Throttling Together

Large systems often use both.

```text
Client
   |
API Gateway
   |
Rate Limiter
   |
Throttler
   |
Services
```

Example:

```text
Rate Limit = 1000/min
Throttling = 50/sec
```

Meaning:

* User cannot exceed 1000 requests/minute.
* Even within that quota, requests are processed at a controlled rate.

---

# Interview Scenario

**Question:** How would you protect a Payment Service from overload?

Answer:

```text
1. Apply rate limiting at API Gateway
   (100 requests/min/user)

2. Apply throttling before Payment Service
   (500 requests/sec total)

3. Queue excess requests

4. Process at safe speed
```

This protects both the user-facing API and the backend service.

---

# One-Line Definitions

### Rate Limiting

> Controls **how many requests** a client can make during a time window. Excess requests are typically rejected.

### Throttling

> Controls **how fast requests are processed** by slowing down, delaying, or queueing traffic to protect system resources.

---

# Easy Interview Memory Trick

```text
Rate Limiting = Count Requests

"How MANY requests are allowed?"
```

Example:

```text
100 requests/minute
```

---

```text
Throttling = Control Speed

"How FAST requests are processed?"
```

Example:

```text
50 requests/second
```

A concise interview answer:

> **Rate limiting decides whether a request is allowed based on request count, while throttling controls the processing speed of allowed requests to prevent resource overload.**
