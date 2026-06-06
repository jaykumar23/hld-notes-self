# Back-of-the-Envelope Estimation (HLD Interview)

This is one of the most important HLD skills.

Interviewers don't expect exact numbers.

They want to see:

✅ Can you estimate scale quickly?

✅ Can you make reasonable assumptions?

✅ Can you calculate storage, traffic, servers, and bandwidth requirements?

---

# What is Back-of-the-Envelope Estimation?

It means:

> Making rough calculations using assumptions to estimate the scale of a system.

Instead of:

```text
Users = 10,432,187
```

You say:

```text
~10 million users
```

The goal is to get the **order of magnitude** right.

---

# Why Interviewers Ask It

Suppose interviewer says:

> Design YouTube.

Before drawing architecture, you need to know:

* How many users?
* How much traffic?
* How much storage?
* How many requests/sec?

Otherwise how will you decide:

* Number of servers
* Database size
* Cache size
* CDN requirements

---

# Common Numbers to Memorize

## Data Sizes

```text
1 KB = 10^3 bytes
1 MB = 10^6 bytes
1 GB = 10^9 bytes
1 TB = 10^12 bytes
1 PB = 10^15 bytes
```

---

## Time

```text
1 minute = 60 sec
1 hour = 3600 sec
1 day = 86400 sec
```

---

## Year

```text
365 days
≈ 3 × 10^7 seconds
```

Very commonly used.

---

# Step-by-Step Approach

Whenever interviewer asks:

```text
Design X
```

Estimate:

### 1. Number of Users

### 2. Daily Active Users

### 3. Requests Per Day

### 4. Requests Per Second

### 5. Storage

### 6. Bandwidth

---

# Example 1: URL Shortener

Like Bitly.

---

## Assumption

```text
100 Million users
```

---

## Daily Active Users

Assume:

```text
10%
```

active.

```text
10 Million DAU
```

---

## Links Created

Assume:

```text
Each user creates 2 links/day
```

Therefore:

```text
20 Million URLs/day
```

---

## Requests Per Second

```text
20 Million/day
```

Divide by:

```text
86400 seconds/day
```

```text
20,000,000 / 86,400
≈ 231
```

Round:

```text
≈ 250 writes/sec
```

---

## Read Traffic

Usually reads are much higher.

Assume:

```text
100× more reads
```

```text
250 × 100
= 25,000 reads/sec
```

---

### Interview Statement

> The system is read-heavy with roughly 25K reads/sec and 250 writes/sec.

This immediately influences:

* Cache design
* DB choice
* Replication strategy

---

# Example 2: Storage Calculation

Suppose each URL record stores:

```text
Original URL = 500 bytes
Metadata = 500 bytes
```

Total:

```text
1 KB per record
```

---

## Daily Storage

```text
20 Million URLs/day
```

```text
20M × 1KB
≈ 20GB/day
```

---

## Yearly Storage

```text
20GB × 365
≈ 7.3TB/year
```

---

### Interview Statement

> We need roughly 7–8 TB of storage annually.

---

# Example 3: Instagram

Assume:

```text
100 Million DAU
```

Each uploads:

```text
1 photo/day
```

---

## Photo Size

Assume:

```text
2 MB
```

---

## Daily Storage

```text
100M × 2MB
```

```text
200,000,000 MB
```

Convert:

```text
200 TB/day
```

---

## Yearly

```text
200 × 365
≈ 73,000 TB
```

```text
≈ 73 PB/year
```

Huge storage requirement.

---

### What Does This Tell Us?

Need:

```text
Object Storage
CDN
Distributed Storage
```

Not a single database.

---

# Example 4: Calculate QPS

Very common.

---

Suppose:

```text
100 Million requests/day
```

Find requests/sec.

---

Formula:

```text
QPS =
Requests per day
--------------
86400
```

---

Calculation:

```text
100,000,000 / 86,400
≈ 1157
```

Round:

```text
≈ 1200 QPS
```

---

# Peak Traffic

Interviewers often ask:

> What about peak traffic?

Traffic isn't uniform.

---

Assume peak is:

```text
5× average
```

---

Average:

```text
1200 QPS
```

Peak:

```text
6000 QPS
```

Design for peak traffic.

---

# Example 5: Cache Estimation

Suppose:

```text
1 Million hot items
```

Each item:

```text
1 KB
```

---

Cache Size:

```text
1M × 1KB
```

```text
≈ 1GB
```

---

### Interview Statement

> A cache of 1–2 GB can store all hot objects.

---

# Example 6: Video Streaming

YouTube-like system.

---

Assume:

```text
10 Million videos
```

Average size:

```text
100 MB
```

---

Storage:

```text
10M × 100MB
```

```text
1 Billion MB
```

```text
≈ 1 PB
```

---

This tells interviewer:

Need:

```text
Distributed storage
CDN
Replication
```

---

# Example 7: Bandwidth Calculation

Suppose:

```text
10,000 users
```

watching videos simultaneously.

Each stream:

```text
5 Mbps
```

---

Bandwidth:

```text
10,000 × 5 Mbps
```

```text
50,000 Mbps
```

```text
50 Gbps
```

---

Huge bandwidth requirement.

Therefore:

```text
Need CDN
```

---

# What Interviewers Evaluate

Not exact numbers.

They evaluate:

### Can you estimate?

### Are assumptions reasonable?

### Can you derive system scale?

---

# Common Assumptions Used in Interviews

## DAU

```text
10%-30% of registered users
```

---

## Peak Traffic

```text
3x - 10x average
```

---

## Read vs Write

Many systems:

```text
90% Reads
10% Writes
```

or

```text
100:1 read/write ratio
```

---

## Replication

Often:

```text
3 replicas
```

So storage becomes:

```text
Raw Storage × 3
```

---

# Interview Example Answer

Suppose interviewer asks:

> Design TinyURL.

Start like this:

> Let's estimate scale first. Assume 100 million users with 10 million daily active users. If each active user creates 2 URLs per day, that's roughly 20 million URLs/day, translating to about 250 writes/sec. Assuming reads are 100× writes, we'll have around 25K reads/sec. If each record is 1 KB, annual storage is approximately 7–8 TB. Since the system is read-heavy, I'll use caching and read replicas.

That sounds like a strong HLD answer.

---

# Quick Formula Sheet

### QPS

```text
Requests/day ÷ 86400
```

---

### Storage

```text
Number of Records × Size
```

---

### Annual Storage

```text
Daily Storage × 365
```

---

### Peak QPS

```text
Average QPS × Peak Factor
```

---

### Bandwidth

```text
Concurrent Users × Data Rate
```

---

# 30-Second Interview Answer

> Back-of-the-envelope estimation is the process of making rough calculations to understand the scale of a system before designing it. In HLD interviews, we estimate users, requests per second, storage, bandwidth, cache size, and peak traffic using reasonable assumptions. The goal is not exact accuracy but to determine the order of magnitude so that architectural decisions such as databases, caching, load balancing, and storage systems can be made appropriately.
