These are **advanced messaging and distributed systems topics** that frequently come up in senior HLD interviews.

---

# 1. Change Data Capture (CDC)

## What is CDC?

CDC means:

> Detect changes happening in a database and publish those changes to other systems.

Instead of:

```text
Application
    |
    +--> Update DB
    |
    +--> Update Search Index
    |
    +--> Update Cache
    |
    +--> Send Event
```

CDC automatically captures database changes.

---

# Why CDC?

Imagine an e-commerce system.

When an order is created:

```sql
INSERT INTO Orders ...
```

Many systems need to know:

```text
Analytics
Search
Cache
Recommendation Engine
Data Warehouse
```

Without CDC:

```text
Order Service
  |
  +--> DB
  +--> Analytics
  +--> Search
  +--> Cache
```

Tightly coupled.

---

# CDC Architecture

```text
Order Service
      |
      v
   Database
      |
      v
 CDC Tool
      |
      v
 Kafka
      |
      +--> Analytics
      +--> Search
      +--> Warehouse
```

---

# How CDC Works

### Example

User creates order:

```sql
INSERT INTO orders
VALUES(1001,'Laptop')
```

CDC detects:

```json
{
  "event":"INSERT",
  "table":"orders",
  "id":1001
}
```

Publishes event.

---

# Two CDC Approaches

---

## 1. Query-Based CDC

Periodically check:

```sql
SELECT *
FROM orders
WHERE updated_at > last_poll_time
```

---

### Problem

```text
High DB load
Slow
Can miss updates
```

Not preferred.

---

## 2. Log-Based CDC (Industry Standard)

Database writes changes to logs.

Examples:

```text
MySQL -> Binlog
PostgreSQL -> WAL
MongoDB -> Oplog
```

CDC tools read logs.

---

Flow:

```text
DB Write
   |
   v
Binlog
   |
   v
Debezium
   |
   v
Kafka
```

No extra DB queries.

Very efficient.

---

# Popular CDC Tools

### Debezium

Most common.

```text
MySQL
Postgres
MongoDB
```

→ Kafka

---

### AWS DMS

Database Migration Service

Supports CDC.

---

# Interview Example

Design:

```text
Amazon Product Catalog
```

Requirement:

```text
Whenever product changes,
update Elasticsearch.
```

Use CDC:

```text
MySQL
  |
 Binlog
  |
 Debezium
  |
 Kafka
  |
 Elasticsearch
```

---

# CDC Advantages

### Decoupling

```text
DB change
 -> Event
```

No direct integrations.

---

### Near Real-Time

Usually milliseconds to seconds.

---

### Reliable

Reads transaction logs.

---

# CDC Interview Summary

```text
Purpose:
Capture DB changes automatically

Best Method:
Log-based CDC

Popular Tool:
Debezium

Use Cases:
- Search Sync
- Analytics
- Data Warehouse
- Event Driven Systems
```

---

# 2. Message Delivery Semantics

This answers:

> How many times can a message be delivered?

---

# Why Needed?

Suppose producer sends:

```text
Order Created
```

Network fails before acknowledgement.

Now producer doesn't know:

```text
Delivered?
Not delivered?
```

It may resend.

This leads to delivery guarantees.

---

# A. At-Most-Once

Message delivered:

```text
0 or 1 time
```

---

Flow:

```text
Send
No Retry
```

---

Example:

```text
Notification
Metrics
Logs
```

Losing some messages acceptable.

---

### Pros

Fast.

---

### Cons

Messages may disappear.

---

Diagram:

```text
Producer
   |
   v
Consumer
```

Failure:

```text
Message Lost
```

---

# B. At-Least-Once

Most common.

Guarantee:

```text
Message never lost
```

But:

```text
Duplicates possible
```

---

Flow:

```text
Send
No ACK
Retry
```

---

Example

```text
Order Created
```

Consumer may receive:

```text
Order 1001
Order 1001
```

twice.

---

Need:

```text
Idempotent Consumers
```

---

### Example

Bad:

```sql
Balance += 100
Balance += 100
```

Duplicate causes issue.

---

Good:

```sql
Process transaction_id once
```

---

# C. Exactly-Once

Guarantee:

```text
No loss
No duplicates
```

---

Very difficult.

Requires:

```text
Transactions
Deduplication
State Tracking
```

---

Used in:

```text
Kafka Streams
Financial Systems
```

---

### Interview Tip

Most real systems use:

```text
At-Least-Once
+
Idempotency
```

because exactly-once is expensive.

---

# Delivery Semantics Table

| Type          | Message Loss | Duplicates |
| ------------- | ------------ | ---------- |
| At-Most-Once  | Possible     | No         |
| At-Least-Once | No           | Possible   |
| Exactly-Once  | No           | No         |

---

# 3. Backpressure Handling

Very important for Kafka and large-scale systems.

---

# What is Backpressure?

Producer is faster than consumer.

---

Example:

Producer generates:

```text
10,000 msgs/sec
```

Consumer processes:

```text
1,000 msgs/sec
```

---

Queue grows:

```text
1000
2000
5000
10000
50000
```

Eventually:

```text
Memory Full
Disk Full
System Crash
```

This overload situation is called:

```text
Backpressure
```

---

# Visualization

```text
Producer
10000/s
    |
    v
 Queue
    |
    v
Consumer
1000/s
```

Messages accumulate.

---

# How to Handle Backpressure?

---

## 1. Scale Consumers

Most common.

```text
Queue
 |
 +--> Consumer1
 +--> Consumer2
 +--> Consumer3
 +--> Consumer4
```

Increase throughput.

---

Example:

```text
1 Consumer = 1000/s

10 Consumers = 10000/s
```

---

## 2. Rate Limiting Producers

Slow down producer.

```text
Producer
   |
   X Too Fast
```

Throttle.

---

Example:

```text
Allowed:
1000 req/sec
```

---

## 3. Bounded Queue

Limit queue size.

```text
Queue Capacity:
10000 Messages
```

If full:

```text
Reject
Drop
Retry
```

---

Example:

```text
RabbitMQ
SQS
Kafka retention limits
```

---

## 4. Pull-Based Consumption

Consumer decides speed.

Instead of:

```text
Server Push
```

Use:

```text
Consumer Pull
```

Kafka works this way.

---

Consumer asks:

```text
Give me 100 messages
```

when ready.

---

## 5. Batch Processing

Process multiple messages together.

Instead of:

```text
1 DB write per message
```

Use:

```text
100 messages
 -> Single DB Write
```

Much faster.

---

## 6. Load Shedding

Drop low-priority traffic.

Example:

```text
Analytics Events
```

Can be discarded.

---

But:

```text
Payments
```

Cannot be discarded.

---

# Kafka Backpressure Example

```text
Producer
    |
    v
 Kafka Topic
    |
    +--> Consumer Group
```

Consumers lag behind.

Kafka tracks:

```text
Current Offset
Consumed Offset
```

Difference is:

```text
Consumer Lag
```

---

Large lag means:

```text
Backpressure
```

---

# Interview Example

Design:

```text
Uber Driver Location Updates
```

Drivers send:

```text
100K updates/sec
```

Consumers process:

```text
10K/sec
```

Solutions:

```text
Add consumers
Batch writes
Throttle producers
Increase partitions
```

---

# Quick HLD Revision

## CDC

```text
Purpose:
Capture DB changes

Best Practice:
Log-Based CDC

Flow:
DB -> Binlog/WAL -> Debezium -> Kafka

Use Cases:
Search Sync
Analytics
Data Warehouse
```

---

## Message Delivery Semantics

```text
At-Most-Once
 -> Fast
 -> Message loss possible

At-Least-Once
 -> No loss
 -> Duplicates possible
 -> Most common

Exactly-Once
 -> No loss
 -> No duplicates
 -> Expensive
```

---

## Backpressure

```text
Producer Faster Than Consumer

Symptoms:
- Queue Growth
- Consumer Lag
- Memory Pressure

Solutions:
- Scale Consumers
- Rate Limiting
- Batch Processing
- Bounded Queues
- Load Shedding
- Pull-Based Consumption
```

### Interview Connection

These three topics often appear together:

```text
Database
   |
   v
CDC (Debezium)
   |
   v
Kafka Topic
   |
   +--> Consumer Group

Delivery:
At-Least-Once

Failure:
Retry + DLQ

High Traffic:
Backpressure Handling
```

This architecture is a very common answer in modern event-driven HLD designs.
