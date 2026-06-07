These are **very common Database Read Scaling topics** in HLD (High-Level Design) interviews.

The interviewer usually wants to know:

> "How will your system handle millions of reads efficiently?"

Let's cover each topic in simple terms and also from an HLD perspective.

---

# 1. Indexing

## Problem

Suppose a Users table has:

```sql
Users
-------------------
id
name
email
city
```

And it contains:

```text
100 Million rows
```

Now a query comes:

```sql
SELECT * FROM Users
WHERE email='abc@gmail.com';
```

Without an index:

Database scans every row.

```text
Row1
Row2
Row3
...
Row100M
```

This is called:

```text
Full Table Scan
```

Very slow.

---

## Solution: Index

Index is a special data structure that helps database find data quickly.

Think of a book.

Without index:

```text
Find chapter "Trees"
Read entire book.
```

With index:

```text
Go to index page.
Jump directly to page 87.
```

---

## Internally

Most databases use:

```text
B+ Tree
```

Example:

```text
           M
         /   \
       G       T
      / \     / \
     A H    N  Z
```

Searching:

```text
O(log N)
```

instead of

```text
O(N)
```

---

## Example

Create index:

```sql
CREATE INDEX idx_email
ON Users(email);
```

Now:

```sql
SELECT * FROM Users
WHERE email='abc@gmail.com';
```

becomes extremely fast.

---

## HLD Interview Answer

For heavy reads:

```text
Create indexes on:
- userId
- email
- createdAt
- frequently searched columns
```

---

# 2. Query Optimization

## Problem

Bad query:

```sql
SELECT *
FROM Orders
WHERE YEAR(created_at)=2025;
```

Database cannot use index efficiently.

It must scan lots of rows.

---

## Better Query

```sql
SELECT *
FROM Orders
WHERE created_at >= '2025-01-01'
AND created_at < '2026-01-01';
```

Now index on:

```text
created_at
```

can be used.

---

## Another Example

Bad:

```sql
SELECT *
```

Good:

```sql
SELECT order_id,status
```

Fetch only required columns.

---

## Why Important

Poor query can make:

```text
1ms query
become
1 second query
```

---

## HLD Interview

Mention:

```text
- Proper indexes
- Avoid full scans
- Use query planner
- Use EXPLAIN
- Fetch only required columns
```

---

# 3. Read Replicas

One of the most important read-scaling concepts.

---

## Problem

Single DB:

```text
          App
           |
      Primary DB
```

Traffic:

```text
100k reads/sec
```

Database overloaded.

---

## Solution

Create replicas.

```text
                Primary
                    |
        ------------------------
        |          |           |
    Replica1   Replica2   Replica3
```

---

## Flow

Writes:

```text
App -> Primary
```

Reads:

```text
App -> Replicas
```

---

## Example

Instagram

```text
Upload Photo
   |
 Primary

View Photo
   |
 Replica
```

Most operations are reads.

---

## Benefits

```text
Read throughput increases
```

Example:

Primary:

```text
10k reads/sec
```

3 replicas:

```text
40k reads/sec
```

approximately.

---

## Replication Lag

Important interview topic.

---

### Problem

User updates profile.

```text
Write -> Primary
```

Replication not yet completed.

Then user reads from replica.

Replica still has old data.

```text
Stale Read
```

---

## HLD Answer

Mention:

```text
Eventual Consistency
Replication Lag
```

For critical reads:

```text
Read from Primary
```

---

# 4. Denormalization

## Normalized Design

```text
Users
Orders
Products
```

Many joins required.

---

Example:

```sql
SELECT *
FROM Orders
JOIN Users
JOIN Products
```

Joins become expensive.

---

## Denormalization

Duplicate data intentionally.

Instead of:

```text
Orders
---------
user_id
```

Store:

```text
Orders
---------
user_id
user_name
```

Now:

```text
No JOIN needed
```

---

## Trade-off

Pros:

```text
Fast Reads
```

Cons:

```text
Duplicate Data
```

Storage increases.

---

## Example

Twitter Feed.

Instead of joining every tweet with user table:

Store:

```text
tweet
username
profile_pic
```

inside feed record.

---

## HLD Interview

Use denormalization when:

```text
Reads >> Writes
```

---

# 5. Materialized Views

Very common in analytics systems.

---

## Problem

Complex query:

```sql
SELECT city,
COUNT(*)
FROM Orders
GROUP BY city;
```

Orders table:

```text
500 Million rows
```

Query expensive.

---

## Materialized View

Precompute result.

```text
CityStats
-----------
Mumbai  1M
Delhi   2M
Pune    500k
```

Stored physically on disk.

---

## Normal View vs Materialized View

### View

```text
Stored Query
```

Runs every time.

---

### Materialized View

```text
Stored Result
```

Runs once.

Reuse result later.

---

## Example

Dashboard

```text
Daily Sales
Top Products
Active Users
```

Precompute every hour.

---

## HLD Interview

Use for:

```text
Analytics
Reports
Dashboards
Heavy Aggregations
```

---

# 6. Connection Pooling

Extremely important.

---

## Problem

Every request creates DB connection.

```text
Client
  |
Open Connection
  |
Query
  |
Close Connection
```

Very expensive.

---

Creating DB connection involves:

```text
TCP handshake
Authentication
Memory allocation
```

---

## Solution

Maintain pool.

```text
       App
         |
 ------------------
 |  Pool of 100   |
 ------------------
         |
       DB
```

---

## Flow

Request 1:

```text
Gets connection #15
```

Request completed.

```text
Returns connection #15
```

Pool reuses it.

---

## Benefits

```text
Less latency
Less CPU
Less DB overhead
```

---

## Example

```text
HikariCP
PgBouncer
c3p0
```

---

## HLD Interview Answer

Always say:

```text
Use connection pooling
to avoid opening a new DB connection
for every request.
```

---

# Complete Read Scaling Progression (Interview Gold)

When reads increase:

```text
Step 1
Proper Indexing
        ↓
Step 2
Query Optimization
        ↓
Step 3
Connection Pooling
        ↓
Step 4
Read Replicas
        ↓
Step 5
Denormalization
        ↓
Step 6
Materialized Views
        ↓
Step 7
Caching (Redis)
```

### One-Line Revision

| Concept              | Purpose                           |
| -------------------- | --------------------------------- |
| Indexing             | Find rows quickly                 |
| Query Optimization   | Execute queries efficiently       |
| Read Replicas        | Scale reads horizontally          |
| Denormalization      | Remove expensive joins            |
| Materialized Views   | Precompute heavy queries          |
| Connection Pooling   | Reuse DB connections              |
| Replication Lag      | Delay between primary and replica |
| Eventual Consistency | Replica may be temporarily stale  |

A strong HLD answer for **"How do you scale database reads?"** is:

```text
1. Add proper indexes
2. Optimize queries
3. Use connection pooling
4. Add read replicas
5. Denormalize hot data
6. Use materialized views for aggregations
7. Add Redis cache for extremely hot reads
```

This is the sequence interviewers typically expect for systems like Instagram, YouTube, Uber, Swiggy, or Amazon.
