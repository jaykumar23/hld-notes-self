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
