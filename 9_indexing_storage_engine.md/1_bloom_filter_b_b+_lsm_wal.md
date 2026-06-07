**WiredTiger** is the **default storage engine of MongoDB**.

Think of MongoDB as having two layers:

```text
MongoDB
   |
   +--> Query Engine
   +--> Replication
   +--> Sharding
   |
   +--> Storage Engine (WiredTiger)
            |
            +--> Disk Storage
            +--> Indexes
            +--> Compression
            +--> Caching
            +--> Concurrency Control
            +--> Recovery
```

When you execute:

```javascript
db.users.insertOne({
   name: "Jay"
})
```

MongoDB itself doesn't directly write bytes to disk.

Instead:

```text
MongoDB
    ↓
WiredTiger
    ↓
Disk
```

WiredTiger handles the actual storage work.

---

# What Does WiredTiger Do?

## 1. Stores Data on Disk

When documents are inserted:

```javascript
{
   _id: 1,
   name: "Jay",
   age: 25
}
```

WiredTiger decides:

```text
Where to store
How to store
When to flush to disk
```

---

## 2. Manages Indexes

Suppose:

```javascript
db.users.createIndex({
   email: 1
})
```

WiredTiger maintains the index using:

```text
B+ Trees
```

Example:

```text
email
 |
 +--> abc@gmail.com
 +--> xyz@gmail.com
 +--> pqr@gmail.com
```

This enables fast lookups.

---

## 3. Write-Ahead Logging (Journal)

Before modifying data:

```text
Write Journal
↓
Update Memory
↓
Flush Later
```

Example:

```text
Insert User
↓
Journal Entry Written
↓
Success Returned
```

If MongoDB crashes:

```text
Read Journal
↓
Replay Operations
↓
Recover Data
```

This is MongoDB's durability mechanism.

---

## 4. Caching (Buffer Pool)

Disk is slow.

WiredTiger keeps frequently used pages in RAM.

```text
Client Query
    |
    v
WiredTiger Cache
    |
 Hit? ----> Return Fast
    |
 Miss
    |
 Disk Read
```

Very similar to:

```text
MySQL Buffer Pool
Postgres Shared Buffers
```

---

## 5. Compression

WiredTiger compresses data automatically.

Without compression:

```text
100 GB Data
```

With compression:

```text
40-60 GB
```

(depending on data)

Common algorithms:

```text
Snappy
zlib
zstd
```

Benefits:

✅ Less storage

✅ Less disk I/O

---

## 6. Concurrency Control

Older MongoDB versions used:

```text
Database Lock
```

Only limited parallelism.

WiredTiger introduced:

```text
Document-Level Locking
```

Example:

```text
User A updates Doc1
User B updates Doc2
```

Both can proceed simultaneously.

Huge performance improvement.

---

# WiredTiger Architecture

```text
                  Client
                     |
                     v
                MongoDB
                     |
                     v
              WiredTiger
                     |
      --------------------------------
      |              |               |
      v              v               v
   Cache         B+ Trees        Journal
      |              |               |
      --------------------------------
                     |
                     v
                Data Files
```

---

# Why MongoDB Switched to WiredTiger?

Before MongoDB 3.2:

```text
MMAPv1
```

Problems:

❌ Global locking

❌ Poor concurrency

❌ Less compression

❌ Less efficient caching

---

WiredTiger solved:

✅ Better concurrency

✅ Compression

✅ Better cache

✅ Better recovery

✅ Better performance

---

# Interview-Level Summary

### What is WiredTiger?

> WiredTiger is MongoDB's default storage engine. It is responsible for storing data on disk, managing B+ Tree indexes, maintaining the write-ahead journal for durability, caching frequently accessed data in memory, compressing data, and providing document-level concurrency control.

---

# Quick HLD Cheat Sheet

```text
MongoDB
   |
   +--> WiredTiger Storage Engine

WiredTiger Provides:
--------------------
✓ B+ Tree Indexes
✓ Journal (WAL)
✓ Cache
✓ Compression
✓ Recovery
✓ Document-Level Locking

Read Optimized:
✓ Fast indexed lookups
✓ Fast range queries

Durability:
✓ Journal/WAL

Not Based On:
✗ LSM Trees (default engine)
```

A common HLD interview comparison is:

```text
MongoDB (WiredTiger)
    = B+ Tree + WAL

Cassandra
    = LSM Tree + WAL + Bloom Filters

MySQL InnoDB
    = B+ Tree + WAL + Buffer Pool

PostgreSQL
    = B+ Tree + WAL + Shared Buffers
```

This comparison often comes up when discussing database selection in system design interviews.
