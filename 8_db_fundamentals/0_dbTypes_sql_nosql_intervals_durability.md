These are **core database concepts** that come up repeatedly in **System Design (HLD) interviews**. The goal is not to become a DBA, but to understand **when to choose what and why**.

---

# 1. Database Types

Databases can be broadly divided into:

## A. Relational Databases (SQL)

Store data in tables.

Example:

### Users Table

| id | name  | email                                     |
| -- | ----- | ----------------------------------------- |
| 1  | John  | [john@gmail.com](mailto:john@gmail.com)   |
| 2  | Alice | [alice@gmail.com](mailto:alice@gmail.com) |

### Orders Table

| id  | user_id | amount |
| --- | ------- | ------ |
| 101 | 1       | 500    |
| 102 | 2       | 1000   |

Relationship:

```
User (1) ------> (Many) Orders
```

Examples:

* MySQL
* PostgreSQL
* Oracle
* SQL Server

Used when:

* Data is structured
* Need joins
* Need strong consistency
* Financial systems

---

## B. NoSQL Databases

Designed for scale and flexibility.

Several types exist.

---

### 1. Key-Value Database

Stores:

```
key -> value
```

Example:

```
user:101 -> {name:"John", age:25}
```

Examples:

* Redis
* DynamoDB

Used for:

* Caching
* Sessions
* User preferences

Complexity:

```
Read = O(1)
Write = O(1)
```

---

### 2. Document Database

Stores JSON-like documents.

Example:

```json
{
  "id": 101,
  "name": "John",
  "skills": ["Java","AWS"]
}
```

Examples:

* MongoDB
* Couchbase

Used when:

* Schema changes frequently
* Product catalogs
* User profiles

---

### 3. Column-Family Database

Stores data by columns instead of rows.

Examples:

* Cassandra
* HBase

Good for:

* Huge datasets
* Time-series
* Analytics

---

### 4. Graph Database

Stores nodes and relationships.

Example:

```
John --> Friend --> Alice
Alice --> Friend --> Bob
```

Examples:

* Neo4j
* Amazon Neptune

Used for:

* Social networks
* Recommendations
* Fraud detection

---

# Interview Answer

If interviewer asks:

### Which DB for Banking?

Answer:

```
SQL Database
(PostgreSQL/MySQL)

Reason:
- ACID transactions
- Strong consistency
```

---

### Which DB for Instagram Feed?

Answer:

```
NoSQL

Reason:
- Massive scale
- Flexible schema
- Fast writes
```

---

# 2. SQL vs NoSQL

One of the most common HLD questions.

---

## SQL

### Characteristics

* Tables
* Fixed schema
* ACID compliant
* Joins supported

Example:

```sql
SELECT *
FROM Users
JOIN Orders
ON Users.id=Orders.user_id;
```

---

### Advantages

Strong consistency

Complex queries

Transactions

Data integrity

---

### Disadvantages

Harder horizontal scaling

Schema changes difficult

---

## NoSQL

### Characteristics

* Flexible schema
* Horizontally scalable
* High throughput

Example:

```json
{
  "userId":101,
  "name":"John"
}
```

---

### Advantages

Easy scaling

Handles huge traffic

Fast writes

---

### Disadvantages

Often weaker consistency

Joins difficult

Complex transactions limited

---

## Quick Comparison

| Feature      | SQL       | NoSQL             |
| ------------ | --------- | ----------------- |
| Schema       | Fixed     | Flexible          |
| Scaling      | Vertical  | Horizontal        |
| Transactions | Strong    | Limited           |
| Consistency  | Strong    | Eventual possible |
| Joins        | Excellent | Difficult         |
| Examples     | MySQL     | MongoDB           |

---

# HLD Rule

If data correctness is critical:

```
SQL
```

If scale is critical:

```
NoSQL
```

Many real systems use both.

Example:

```
Instagram

MySQL -> User metadata

Redis -> Cache

Cassandra -> Feed storage
```

---

# 3. ACID Transactions

ACID ensures database reliability.

---

## A = Atomicity

Transaction happens completely or not at all.

Example:

Transfer ₹100.

Before:

```
A = 1000
B = 500
```

Steps:

```
A -= 100
B += 100
```

If server crashes after first step:

```
A = 900
B = 500
```

Wrong.

Atomicity ensures:

```
Either both happen
or neither happens
```

---

## C = Consistency

Database always remains valid.

Rule:

```
A + B = 1500
```

After transaction:

```
900 + 600 = 1500
```

Still valid.

---

## I = Isolation

Concurrent transactions should not interfere.

---

Without Isolation

T1:

```
Withdraw 100
```

T2:

```
Read balance
```

T2 may see partial updates.

Isolation prevents this.

---

## D = Durability

Once committed:

```
Data survives crash
```

Even if power goes off.

Database recovers committed data.

---

## ACID Example

Bank Transfer:

```
BEGIN

A -= 100
B += 100

COMMIT
```

After commit:

```
Guaranteed permanent
```

---

# 4. Database Internals

Interviewers sometimes ask:

> What happens when INSERT is executed?

---

## Step 1: Query Arrives

```sql
INSERT INTO users VALUES(1,'John');
```

Client sends query.

---

## Step 2: Parser

Checks syntax.

```sql
INSERT INTO ...
```

Valid?

If no:

```
Syntax Error
```

---

## Step 3: Optimizer

Chooses best execution plan.

For SELECT:

```
Use index?
Use full scan?
```

---

## Step 4: Storage Engine

Actual component storing data.

Examples:

```
MySQL -> InnoDB
Postgres -> Heap Storage
```

---

## Step 5: Write Ahead Log (WAL)

Before writing data:

```
Log entry written
```

This is crucial.

Example:

```
INSERT user 101
```

Log stores:

```
Operation details
```

---

## Step 6: Memory Update

Data written into:

```
Buffer Pool
```

(in-memory pages)

Fast operation.

---

## Step 7: Disk Write Later

Background process flushes data.

```
RAM -> Disk
```

---

### Simplified Flow

```
Query
  ↓
Parser
  ↓
Optimizer
  ↓
WAL
  ↓
Buffer Pool
  ↓
Disk
```

---

# 5. How Databases Guarantee Durability

One of the favorite HLD questions.

Question:

> If data is first written to memory, what if machine crashes?

Answer:

```
Write-Ahead Logging (WAL)
```

---

## Problem

Suppose:

```sql
INSERT USER 101
```

Database stores only in RAM.

Crash occurs.

Data lost.

---

## Solution: WAL

Before modifying data:

Database writes operation to log file.

```
WAL Disk
```

Example:

```
INSERT USER 101
```

stored in WAL.

---

### Flow

```
Write request
     ↓
WAL disk write
     ↓
Commit success
     ↓
Memory update
     ↓
Disk flush later
```

---

## Crash Happens

Suppose crash occurs.

Data page not written yet.

But WAL exists.

Recovery process reads WAL.

```
Replay operations
```

Data restored.

---

## Recovery Example

WAL contains:

```
INSERT USER 101
UPDATE USER 102
DELETE USER 103
```

Database restarts.

Reads WAL.

Replays operations.

Database becomes consistent.

---

# Checkpointing

If WAL grows forever:

```
Huge recovery time
```

Database periodically creates checkpoints.

Checkpoint:

```
Current memory pages
→ flushed to disk
```

Then old WAL can be discarded.

---

### Recovery Process

```
Disk Data
   +
Checkpoint
   +
WAL
   =
Latest State
```

---

# Real Interview Example

### Q: Why is MySQL durable?

Answer:

```
MySQL InnoDB uses:
1. Redo Log (WAL)
2. Buffer Pool
3. Checkpoints

Transaction is considered committed only after redo log is safely persisted.

During crash recovery, redo logs are replayed to restore committed transactions.
```

---

# HLD Interview Summary Sheet

## Database Types

* SQL
* Key-Value
* Document
* Column-Family
* Graph

---

## SQL vs NoSQL

SQL:

* Strong consistency
* Joins
* ACID

NoSQL:

* Scale
* Flexible schema
* High throughput

---

## ACID

* Atomicity → all or nothing
* Consistency → valid state
* Isolation → no interference
* Durability → survives crash

---

## Database Internals

```
Query
 → Parser
 → Optimizer
 → WAL
 → Buffer Pool
 → Disk
```

---

## Durability

```
WAL / Redo Log
Checkpointing
Crash Recovery
```

These concepts are enough to confidently answer most database-related questions in HLD interviews at companies like Amazon, Flipkart, Uber, Swiggy, Walmart, PayPal, and similar product companies.



This is a key concept in **database internals** and is often asked in HLD interviews.

Let's use PostgreSQL/MySQL InnoDB style architecture.

---

# First Understand: Why Not Write Directly To Disk?

Disk I/O is slow.

Approximate speeds:

| Storage   | Latency |
| --------- | ------- |
| CPU Cache | ~1 ns   |
| RAM       | ~100 ns |
| SSD       | ~100 µs |
| HDD       | ~10 ms  |

Disk is **thousands to millions of times slower** than memory.

If every update waited for a disk write:

```sql
UPDATE users
SET name='Jay'
WHERE id=1;
```

the database would become very slow.

So databases use:

```
RAM (fast)
+
WAL Log (durability)
+
Background Disk Flush
```

---

# Database Components

```
                Client
                   |
                   v
             SQL Engine
                   |
                   v
        +-------------------+
        |   Buffer Pool     |
        |  (RAM Pages)      |
        +-------------------+
                   |
                   v
          WAL / Redo Log
                   |
                   v
             Disk Storage
```

---

# Step 1: Read Page Into Memory

Suppose table page on disk contains:

```
User 1 -> Balance = 1000
```

Disk:

```
Page P1
```

When query arrives:

```sql
UPDATE accounts
SET balance=800
WHERE id=1;
```

Database loads page into RAM.

```
Disk Page P1
      |
      v
Buffer Pool Page P1
```

---

# Step 2: Write WAL First

Before changing actual data page:

Database creates log entry.

```
UPDATE account 1
1000 -> 800
```

Written to WAL.

```
WAL File
---------
LSN 101:
Account 1
1000 -> 800
---------
```

Flushed to disk immediately.

This guarantees durability.

---

# Step 3: Commit Success

After WAL reaches disk:

```sql
COMMIT;
```

Database tells client:

```
Transaction successful
```

Notice:

**Actual table page may still not be on disk.**

Only WAL is guaranteed on disk.

---

# Step 4: Memory Update (Your Question)

Now database updates page inside Buffer Pool.

Before:

```
Buffer Pool Page P1

Account1 = 1000
```

After:

```
Buffer Pool Page P1

Account1 = 800
```

This page is now called:

```
Dirty Page
```

Meaning:

```
RAM version != Disk version
```

---

## What is a Dirty Page?

Disk:

```
Account1 = 1000
```

RAM:

```
Account1 = 800
```

Different values.

Therefore:

```
Dirty Page
```

Needs flushing later.

---

# Step 5: Continue Serving Requests

Now all future reads can use RAM.

```sql
SELECT balance
FROM accounts
WHERE id=1;
```

returns:

```
800
```

from memory.

No disk access required.

Very fast.

---

# Step 6: Background Flush (Checkpoint)

Every few seconds/minutes:

Background thread wakes up.

Examples:

```
Postgres -> Checkpointer
MySQL -> Page Cleaner
```

It scans dirty pages.

```
Dirty Page P1
```

and writes them to disk.

---

Before Flush

Disk:

```
1000
```

RAM:

```
800
```

---

After Flush

Disk:

```
800
```

RAM:

```
800
```

Page becomes clean.

---

# What If Crash Happens Before Flush?

Suppose:

```
WAL written ✓
Page flush not done ✗
```

Then power failure occurs.

---

Disk page:

```
1000
```

WAL:

```
1000 -> 800
```

exists on disk.

---

Database restarts.

Recovery process runs.

Reads WAL:

```
LSN101:
1000 -> 800
```

Replays change.

Disk becomes:

```
800
```

Data recovered.

This is why WAL is sufficient for durability.

---

# Timeline Example

### T0

Disk:

```
1000
```

RAM:

```
1000
```

---

### T1 Update Query

```sql
UPDATE balance=800
```

---

### T2 WAL Written

Disk:

```
WAL:
1000 -> 800
```

---

### T3 Commit

Client gets:

```
SUCCESS
```

---

### T4 Memory Updated

RAM:

```
800
```

Disk page:

```
1000
```

Dirty page exists.

---

### T5 Crash

No problem.

Recovery uses WAL.

---

### T6 Restart

Replay WAL.

Disk:

```
800
```

---

### T7 Normal Operation

Background flush writes page permanently.

---

# Interview One-Liner

**Data pages are first updated in the Buffer Pool (RAM). These become dirty pages. A background process later flushes dirty pages to disk. Durability is guaranteed because the WAL/redo log was already persisted before the transaction committed.**
