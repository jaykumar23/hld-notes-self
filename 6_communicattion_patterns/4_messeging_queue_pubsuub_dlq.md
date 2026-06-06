These are **core distributed system concepts** and appear in almost every HLD interview (Uber, Amazon, Flipkart, Walmart, Swiggy, Zomato, Microsoft, etc.).

---

# 1. Message Queues

## What Problem Do They Solve?

Without a queue:

```text
Client
  |
  v
Order Service
  |
  +--> Payment Service
  |
  +--> Email Service
  |
  +--> Inventory Service
```

Order service waits for everything.

If Email service is slow:

```text
Order Creation = Slow
```

If Email service crashes:

```text
Order Creation = Failed
```

This is called **tight coupling**.

---

## Solution: Message Queue

Put a queue between services.

```text
Producer                Consumer

Order Service ----> Queue ----> Email Service
```

Order service only sends a message.

It doesn't care when Email service processes it.

---

## Real Example

User places order.

Instead of:

```text
Create Order
Send Email
Update Inventory
Generate Invoice
```

Synchronously,

do:

```text
Create Order
Push Message to Queue
Return Success
```

Background workers process messages later.

---

## Components

### Producer

Creates messages.

```text
Order Service
```

Example:

```json
{
  "orderId": 123,
  "userId": 456
}
```

---

### Queue

Stores messages.

```text
FIFO Buffer
```

Example:

```text
Msg1
Msg2
Msg3
```

---

### Consumer

Reads messages.

```text
Email Service
```

Consumes:

```json
{
  "orderId":123
}
```

and sends email.

---

# Queue Flow

```text
Producer
    |
    v
+---------+
| Queue   |
+---------+
    |
    v
Consumer
```

---

# Why Use Message Queues?

## 1. Decoupling

Services become independent.

```text
Order Service
    |
    v
 Queue
    |
    v
 Email Service
```

Email crashes?

Order creation still works.

---

## 2. Reliability

Messages are stored.

Consumer can process later.

---

## 3. Scalability

Add more consumers.

```text
Queue
 |
 +--> Worker 1
 |
 +--> Worker 2
 |
 +--> Worker 3
```

Higher throughput.

---

## 4. Load Leveling

Traffic spikes handled smoothly.

Example:

Black Friday:

```text
100K orders/min
```

Workers process gradually.

Queue acts as buffer.

---

# Popular Queue Systems

### RabbitMQ

```text
Traditional MQ
```

Good routing.

---

### Kafka

```text
Distributed Event Streaming
```

Very high throughput.

---

### Amazon SQS

```text
Managed Queue
```

No infrastructure management.

---

# Queue Interview Diagram

```text
Users
   |
   v
Order Service
   |
   v
 Message Queue
   |
   +--> Email Service
   |
   +--> Invoice Service
   |
   +--> Inventory Service
```

---

# Important Interview Question

## Can Messages Be Lost?

Depends.

### At-most-once

```text
May lose messages
No duplicates
```

---

### At-least-once

```text
No loss
Duplicates possible
```

Most common.

---

### Exactly-once

```text
No loss
No duplicates
```

Very difficult.

Expensive.

---

# 2. Publish-Subscribe (Pub/Sub)

Pub/Sub is a messaging pattern.

Instead of:

```text
Producer -> One Consumer
```

we have:

```text
Producer -> Many Consumers
```

---

# Real Life Example

YouTube uploads video.

Subscribers receive notification.

```text
One Publisher
Many Subscribers
```

---

# Architecture

```text
                +--> Email Service
                |
Publisher --> Topic --> Analytics Service
                |
                +--> Recommendation Service
                |
                +--> Notification Service
```

---

# Key Concepts

## Publisher

Produces event.

```text
Order Created
```

---

## Topic

Logical channel.

```text
order-created
```

---

## Subscriber

Receives events.

```text
Email Service
Inventory Service
Analytics Service
```

---

# Example

Order Created Event:

```json
{
  "orderId": 1001
}
```

Published to:

```text
order-created topic
```

Subscribers receive it.

---

# Difference from Queue

## Queue

One message → One consumer

```text
Producer
   |
 Queue
   |
Consumer
```

Only one worker gets it.

---

## Pub/Sub

One message → Many consumers

```text
Publisher
   |
 Topic
   |
 +--> Consumer A
 |
 +--> Consumer B
 |
 +--> Consumer C
```

Everyone gets a copy.

---

# Queue vs Pub/Sub

| Feature          | Queue             | Pub/Sub             |
| ---------------- | ----------------- | ------------------- |
| Consumers        | One               | Many                |
| Goal             | Work Distribution | Event Distribution  |
| Example          | Image Processing  | Order Created Event |
| Message Delivery | One Consumer      | All Subscribers     |

---

# Interview Example

### Queue

```text
Video Processing
```

Only one worker should process a video.

---

### Pub/Sub

```text
Order Created
```

Many services interested.

```text
Email
Analytics
Inventory
Fraud Detection
```

---

# Kafka and Pub/Sub

Kafka commonly implements Pub/Sub.

```text
Topic
   |
   +--> Consumer Group A
   +--> Consumer Group B
```

Each group gets copy.

Inside group:

```text
Workers share load.
```

Best of both worlds.

---

# 3. Dead Letter Queue (DLQ)

Very important interview topic.

---

# Problem

Consumer fails to process message.

Example:

```json
{
  "orderId":"INVALID"
}
```

Consumer crashes.

---

# What Happens?

Queue retries.

```text
Attempt 1 -> Fail
Attempt 2 -> Fail
Attempt 3 -> Fail
```

Still fails.

Now what?

---

Without DLQ

```text
Queue stuck forever
```

Bad.

---

# Solution: Dead Letter Queue

Failed messages moved to separate queue.

```text
Main Queue
     |
     v
Processing
     |
     X
     |
     v
Dead Letter Queue
```

---

# Flow

```text
Producer
    |
    v
 Main Queue
    |
    v
 Consumer
    |
    X Failure
    |
    v
 DLQ
```

---

# Example

Order Event:

```json
{
 "orderId": null
}
```

Invalid message.

After 5 retries:

```text
Move to DLQ
```

---

# Why DLQ?

## Prevent Infinite Retries

Without DLQ:

```text
Fail
Retry
Fail
Retry
Forever
```

---

## Debugging

Developers inspect bad messages.

```text
Why failing?
```

Easy investigation.

---

## Reliability

Good messages continue processing.

Bad message isolated.

---

# Interview Scenario

Design:

```text
Uber Ride Booking
```

Events:

```text
Ride Created
Payment Completed
Driver Assigned
```

Payment event malformed.

Instead of blocking queue:

```text
Move to DLQ
```

Operations team fixes later.

---

# Retry + DLQ Pattern

Most common design.

```text
Producer
   |
   v
Main Queue
   |
   v
Consumer
   |
   +--> Success
   |
   +--> Retry Queue
            |
            v
         Retry 3 Times
            |
            v
           DLQ
```

---

# HLD Interview Summary

```text
Message Queue
-------------
Purpose:
Decouple services

Examples:
RabbitMQ, Kafka, SQS

Benefits:
- Async processing
- Reliability
- Scalability
- Buffer traffic spikes

------------------------------------------------

Publish-Subscribe
-----------------
Purpose:
One event -> Many consumers

Examples:
Kafka Topics
Google Pub/Sub
SNS

Benefits:
- Event-driven architecture
- Loose coupling
- Multiple subscribers

------------------------------------------------

Dead Letter Queue (DLQ)
-----------------------
Purpose:
Store failed messages

Benefits:
- Avoid infinite retries
- Debug failures
- Improve reliability

Flow:
Main Queue -> Retry -> DLQ
```

### Easy Interview Rule

```text
Need background processing?
→ Message Queue

Need one event consumed by many services?
→ Pub/Sub

Need to handle permanently failed messages?
→ DLQ
```

A lot of modern systems combine all three:

```text
Order Service
      |
      v
Kafka Topic (Pub/Sub)
      |
      +--> Email Service
      +--> Inventory Service
      +--> Analytics Service

Each service:
      |
      v
Retry Logic
      |
      v
Dead Letter Queue
```

This is a very common production-grade architecture discussed in HLD interviews.
