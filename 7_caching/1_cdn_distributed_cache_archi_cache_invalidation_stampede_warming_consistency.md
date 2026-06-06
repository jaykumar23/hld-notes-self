These topics frequently appear in HLD interviews for systems like Netflix, YouTube, Instagram, Amazon, Swiggy, etc.

---

# 1. Content Delivery Network (CDN)

## What is a CDN?

A CDN is a network of geographically distributed servers that cache static content closer to users.

Without CDN:

```text
User (Mumbai)
      ↓
Server (USA)
```

Every request travels thousands of kilometers.

---

With CDN:

```text
User (Mumbai)
      ↓
CDN Edge Server (Mumbai)
      ↓
Origin Server (USA) [only if needed]
```

Much faster.

---

## Example: Instagram Image

Without CDN:

```text
Mumbai User
    ↓
US Server
    ↓
Image
```

Latency:

```text
300-500 ms
```

---

With CDN:

```text
Mumbai User
    ↓
Mumbai CDN
    ↓
Image
```

Latency:

```text
20-50 ms
```

---

## CDN Flow

### Cache Hit

```text
User
 ↓
CDN
 ↓
Image Found
 ↓
Return Image
```

---

### Cache Miss

```text
User
 ↓
CDN
 ↓
Not Found
 ↓
Origin Server
 ↓
Fetch Content
 ↓
Store in CDN
 ↓
Return
```

---

## What is Cached?

Typically:

✅ Images

✅ CSS

✅ JavaScript

✅ Videos

✅ PDFs

Usually NOT:

❌ Frequently changing user data

❌ Banking transactions

---

## Popular CDNs

* Cloudflare
* Akamai Technologies
* Amazon Web Services (CloudFront)

---

# Interview Answer

> A CDN caches static content at edge locations close to users, reducing latency, bandwidth costs, and load on origin servers.

---

# 2. Distributed Cache Architecture

When one cache server becomes insufficient:

```text
Application
     ↓
 Redis
```

Problems:

* Memory limit
* Single point of failure

---

## Distributed Cache

```text
Application
      ↓
 Redis Cluster
 ├── Node 1
 ├── Node 2
 ├── Node 3
 └── Node 4
```

---

## Sharding

Data distributed across nodes.

Example:

```text
User:100 → Node1
User:101 → Node2
User:102 → Node3
```

Using:

```text
hash(key) % N
```

---

## Replication

```text
Primary
   ↓
Replica
   ↓
Replica
```

Provides failover.

---

## Interview Keywords

✅ Sharding

✅ Consistent Hashing

✅ Replication

✅ Failover

---

# 3. Cache Invalidation

One of the hardest distributed systems problems.

The challenge:

```text
Database = New Data
Cache = Old Data
```

Users see stale information.

---

## Example

Product price:

```text
DB = ₹999
Cache = ₹899
```

Incorrect result.

---

## Solution 1: TTL

Set expiration.

```text
TTL = 10 minutes
```

After expiry:

```text
Cache removed
```

---

### Pros

Simple.

### Cons

Stale data exists until expiration.

---

## Solution 2: Explicit Invalidation

Whenever data changes:

```text
Update DB
      ↓
Delete Cache
```

Next read:

```text
Cache Miss
→ DB Read
→ Cache Rebuild
```

Most common interview answer.

---

## Example

```text
PUT /user/123
```

```text
Update DB
Delete Cache(user:123)
```

---

# Interview Answer

> After updating the database, I would invalidate the corresponding cache entry and let future reads repopulate it.

---

# 4. Cache Stampede

Also called:

### Thundering Herd Problem

---

Imagine:

```text
Popular Product
```

Cache expires.

At same moment:

```text
100,000 users request it
```

---

Result:

```text
100,000 DB Queries
```

Database overload.

---

## Diagram

```text
Cache Expired
      ↓
100k Requests
      ↓
Database
```

Boom 💥

---

## Solution 1: Mutex Lock

Only one request rebuilds cache.

```text
Request 1
   ↓
Acquire Lock
   ↓
Read DB
   ↓
Update Cache
```

Others wait.

---

```text
Request 2
Request 3
Request 4
```

Read updated cache later.

---

## Solution 2: Probabilistic Expiration

Refresh before expiry.

---

## Solution 3: Stale While Revalidate

Serve old cache temporarily.

```text
Return Old Data
      ↓
Refresh in Background
```

Popular in CDNs.

---

# Interview Answer

> To prevent cache stampede, I'd use request coalescing with a mutex lock so only one request rebuilds the cache while others wait or use stale data.

---

# 5. Cache Warming

## Problem

After deployment:

```text
Cache Empty
```

Millions of requests:

```text
All hit DB
```

Huge spike.

---

## Cache Warming

Preload frequently used data.

Before traffic arrives:

```text
Load Top Products
Load Popular Videos
Load Trending Posts
```

Into cache.

---

## Example

Netflix launches new server.

Before opening traffic:

```text
Top Movies
Top TV Shows
Trending Content
```

Loaded into cache.

---

## Benefits

✅ Faster startup

✅ Fewer cache misses

✅ Lower DB load

---

# Interview Answer

> Cache warming preloads frequently accessed data into cache before traffic arrives, reducing cold-start latency and database spikes.

---

# 6. Cache Consistency

## Problem

Database and cache can diverge.

```text
DB    = New Value
Cache = Old Value
```

---

### Example

User updates profile:

```text
Name = Rahul
```

DB updated.

Cache still contains:

```text
Name = Raj
```

---

Users see incorrect information.

---

# Consistency Models

## Strong Consistency

```text
Read always sees latest write
```

Usually:

```text
Write Cache
Write DB
```

or

```text
Write DB
Invalidate Cache
```

before acknowledging success.

---

### Pros

Accurate.

### Cons

Slower.

---

## Eventual Consistency

```text
DB updated now
Cache updated later
```

Temporary mismatch allowed.

---

Example:

Instagram likes.

```text
100 Likes
```

One user sees:

```text
99 Likes
```

for a few seconds.

Acceptable.

---

### Pros

High performance.

### Cons

Temporary stale data.

---

# Strong vs Eventual

| Feature    | Strong  | Eventual     |
| ---------- | ------- | ------------ |
| Accuracy   | High    | Medium       |
| Speed      | Lower   | Higher       |
| Complexity | Higher  | Lower        |
| Use Cases  | Banking | Social Media |

---

# Complete Interview Architecture

```text
Users
   ↓
CDN
   ↓
Load Balancer
   ↓
Application Servers
   ↓
Redis Cluster
   ↓
Database
```

### Techniques Used

* CDN for static content
* Distributed Redis cache
* Cache-aside strategy
* TTL + explicit invalidation
* Mutex locks for stampede prevention
* Cache warming for popular data
* Eventual consistency where acceptable

---

# 60-Second HLD Answer

> "I would place a CDN in front of the application to cache static assets globally. For dynamic data, I'd use a distributed Redis cache with sharding and replication. I'd follow a cache-aside strategy, invalidate cache entries on updates, use TTLs for cleanup, prevent cache stampedes using locks or stale-while-revalidate, warm the cache with hot data during deployments, and choose either strong or eventual consistency based on business requirements."
