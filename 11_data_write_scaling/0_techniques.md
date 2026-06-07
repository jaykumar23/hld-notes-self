These are **very important write-scaling concepts** in HLD/System Design interviews.

A common interview progression is:

**Single DB → Vertical Scaling → Replication → Partitioning/Sharding → Multi-Region DB**

---

# 1. Vertical Partitioning

## What is it?

Split a table **by columns**.

Instead of keeping all columns in one table/database, divide them into multiple tables.

### Before

Users Table

| UserID | Name | Email | Address | ProfilePic | Bio |
| ------ | ---- | ----- | ------- | ---------- | --- |

Some columns are accessed frequently while others are rarely used.

---

### After Vertical Partitioning

### User Basic

| UserID | Name | Email |
| ------ | ---- | ----- |

### User Profile

| UserID | Address | ProfilePic | Bio |
| ------ | ------- | ---------- | --- |

---

## Why do this?

Most requests need:

```sql
SELECT Name, Email
```

They don't need:

```sql
ProfilePic
Bio
Address
```

Reading less data = faster.

---

## Benefits

### Smaller rows

Instead of reading:

```text
Name + Email + Address + Bio + Photo
```

Read only:

```text
Name + Email
```

---

### Better cache utilization

More rows fit in memory.

---

### Less disk I/O

Fewer pages need loading.

---

## Drawback

Need joins.

```sql
UserBasic
JOIN UserProfile
```

More complexity.

---

## Interview Example

Facebook:

```text
Frequently used user data
→ separate table

Rare profile data
→ separate table
```

---

# 2. Sharding

The most important write-scaling technique.

---

## Problem

One database server can handle only limited writes.

Suppose:

```text
1 DB

Max Writes:
50,000 writes/sec
```

Traffic grows:

```text
500,000 writes/sec
```

Single DB dies.

---

## Solution

Split data across multiple databases.

---

### Before

```text
         Users
           |
           v
       DB-1
```

---

### After

```text
Users 1-1M  -> DB-1

Users 1M-2M -> DB-2

Users 2M-3M -> DB-3
```

Now writes spread across servers.

---

## Example

10 shards

Each handles:

```text
50K writes/sec
```

Total:

```text
500K writes/sec
```

---

## Types of Sharding

---

# A. Range Based Sharding

Split by ranges.

### Example

```text
UserID 1-1M     -> DB1

UserID 1M-2M    -> DB2

UserID 2M-3M    -> DB3
```

---

### Problem

Hotspot.

If new users always get increasing IDs:

```text
Newest writes
→ DB3 only
```

DB3 overloaded.

---

# B. Hash Based Sharding

Most common.

---

Use:

```text
hash(UserID) % N
```

Example:

```text
UserID = 123

hash(123)%4

= shard 3
```

---

Distribution becomes:

```text
DB1
DB2
DB3
DB4
```

Almost evenly.

---

### Benefit

Balanced traffic.

---

### Drawback

Range queries difficult.

Example:

```sql
SELECT *
WHERE UserID BETWEEN 1000 AND 2000
```

Data may exist everywhere.

---

# C. Geo Sharding

Shard by region.

---

```text
India Users -> India DB

US Users -> US DB

Europe Users -> Europe DB
```

---

Useful for:

```text
Netflix
Uber
Amazon
```

---

# Sharding Flow

Request:

```text
UserID=5000
```

Router:

```text
hash(5000)%4
```

Result:

```text
Shard 2
```

Write goes directly to Shard 2.

---

# Benefits

### Horizontal write scaling

Add more servers.

---

### Higher throughput

More writes/sec.

---

### Smaller indexes

Each shard stores less data.

---

### Better performance

Queries become faster.

---

# Drawbacks

### Cross-shard joins

Hard.

---

### Transactions difficult

Across multiple shards.

---

### Rebalancing painful

When adding new shards.

---

# 3. Sharding vs Partitioning

Interview favorite.

---

## Partitioning

One database.

Split data internally.

```text
DB Server
 ├── Partition A
 ├── Partition B
 └── Partition C
```

Still one server.

---

## Sharding

Multiple databases.

```text
DB1
DB2
DB3
DB4
```

Different machines.

---

## Partitioning Example

Inside PostgreSQL:

```sql
PARTITION BY RANGE
```

Same server.

---

## Sharding Example

```text
Customer Data

DB1
DB2
DB3
```

Different servers.

---

## Quick Interview Answer

### Partitioning

```text
Logical split
Usually inside one DB
Improves query performance
```

### Sharding

```text
Physical split
Across multiple servers
Improves write scalability
```

---

# 4. Data Compression

Another write-scaling optimization.

---

## Problem

Storage grows.

Example:

```text
100 TB logs
```

Huge cost.

---

## Solution

Compress data.

---

### Before

```text
AAAAAAAAAA
```

10 bytes

---

### After

```text
A x 10
```

Much smaller.

---

## Benefits

### Less storage

```text
100 TB -> 30 TB
```

---

### Faster disk reads

Less data transferred.

---

### Better network efficiency

Replication becomes faster.

---

## Compression Types

### Row Compression

Compress row.

---

### Column Compression

Very common in analytics.

Example:

```text
Age
25
25
25
25
25
```

Compresses extremely well.

Used in:

```text
Snowflake
BigQuery
Redshift
```

---

## Drawback

Need CPU to compress/decompress.

---

## Interview Use Case

Analytics databases.

```text
Huge data
Mostly reads
```

Compression saves money.

---

# 5. Multi-Region Databases

Used by global products.

---

## Problem

Users are worldwide.

Example:

```text
India
USA
Europe
Australia
```

Single US database causes latency.

---

### User in India

Request:

```text
India → USA DB
```

May take:

```text
200-300 ms
```

Too slow.

---

## Solution

Deploy databases in multiple regions.

---

### Architecture

```text
India Region
    DB

US Region
    DB

Europe Region
    DB
```

Users connect to nearest region.

---

## Benefits

### Low latency

India user:

```text
India DB
```

Fast.

---

### High availability

US outage?

India DB still works.

---

### Disaster recovery

Region failure won't kill system.

---

# Multi-Region Write Challenges

Hardest part.

---

## Conflict Example

India:

```text
Update Name = Jay
```

At same time

USA:

```text
Update Name = Kumar
```

Who wins?

---

Need:

### Last Write Wins

or

### Conflict Resolution

or

### Consensus Algorithms

```text
Raft
Paxos
```

---

# Common Architectures

## Active-Passive

```text
US Primary

India Replica
Europe Replica
```

Writes:

```text
US only
```

Simple.

---

## Active-Active

```text
US
India
Europe

All accept writes
```

More scalable.

Much harder.

---

# Interview Summary (1-Minute Revision)

| Concept                  | Simple Meaning                | Why Used                        |
| ------------------------ | ----------------------------- | ------------------------------- |
| Vertical Partitioning    | Split table by columns        | Smaller rows, faster queries    |
| Sharding                 | Split data across DB servers  | Increase write capacity         |
| Partitioning             | Split data inside one DB      | Better query performance        |
| Sharding vs Partitioning | Physical vs Logical split     | Scalability discussion          |
| Data Compression         | Store data in smaller form    | Save storage & bandwidth        |
| Multi-Region DB          | Databases in multiple regions | Low latency & high availability |

### Interview One-Liner

**"For write scaling, the most important technique is sharding. Vertical partitioning reduces row size, compression reduces storage costs, and multi-region databases reduce latency globally. Partitioning helps query performance, while sharding actually increases write throughput by distributing writes across multiple database servers."**
