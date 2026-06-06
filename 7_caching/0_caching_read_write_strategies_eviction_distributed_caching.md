These are very common HLD interview topics. Interviewers usually expect you to know **why** caching is used, **where** it fits in the architecture, and the trade-offs.

---

# 1. What is Caching?

### Simple Definition

A **cache** is a temporary storage layer that keeps frequently accessed data in memory so that future requests can be served faster.

Instead of:

```
User → Application → Database
```

we do:

```
User → Application → Cache
                 ↓ (cache miss)
              Database
```

---

## Why do we need caching?

Databases are relatively slow.

Example:

| Operation         | Time      |
| ----------------- | --------- |
| RAM Cache (Redis) | 1-5 ms    |
| Database Query    | 50-500 ms |

If 1 million users request the same product details:

Without cache:

```
1M requests → DB
```

Database becomes overloaded.

With cache:

```
1M requests → Redis Cache
```

Database load drops dramatically.

---

## Example: Amazon Product Page

Product ID = 101

First request:

```
User requests Product 101

Cache ❌ Miss

→ Read from DB
→ Store in Cache
→ Return response
```

Second request:

```
User requests Product 101

Cache ✅ Hit

→ Return directly from Cache
```

No database call needed.

---

## Cache Hit vs Cache Miss

### Cache Hit

Data found in cache.

```
Request → Cache → Data Found
```

Fast.

---

### Cache Miss

Data not found in cache.

```
Request → Cache
          ↓
        Not Found
          ↓
        Database
          ↓
      Save to Cache
          ↓
       Response
```

Slower.

---

## Common Cache Technologies

* Redis
* Memcached
* Browser Cache
* CDN Cache

For HLD interviews, Redis is the most commonly discussed cache.

---

# 2. Read-Through Cache

In Read-Through caching, the application first checks the cache.

If data is absent, it fetches from DB and stores it in cache.

---

## Flow

```
Request
   ↓
Cache
   ↓
Hit? ------ Yes ----> Return Data
   ↓ No
Database
   ↓
Store in Cache
   ↓
Return Data
```

---

## Example

Get user profile:

```javascript
function getUser(id) {
    user = cache.get(id);

    if(user)
        return user;

    user = db.get(id);

    cache.set(id, user);

    return user;
}
```

---

## Real World Example

Instagram profile page.

```
Request Profile
     ↓
Redis
     ↓
Not Found
     ↓
MySQL
     ↓
Store in Redis
     ↓
Return
```

---

## Advantages

### Reduces DB load

Frequently accessed data stays in cache.

### Faster reads

Most requests never hit DB.

### Easy to implement

Very common interview answer.

---

## Disadvantage

First request is slow.

```
Cache Miss
→ DB Read
→ Cache Update
```

---

# 3. Write-Through Cache

Problem:

Suppose user updates profile.

```
DB Updated
Cache Still Old
```

Now cache contains stale data.

---

### Write-Through Solution

Whenever data is written:

```
Write Cache
Write DB
```

Both are updated together.

---

## Flow

```
User Update
      ↓
Cache Update
      ↓
Database Update
      ↓
Success
```

---

## Example

User changes name:

Old:

```
Name = John
```

New:

```
Name = John Smith
```

Write-through:

```
Update Cache
Update DB
Return Success
```

Now both contain latest value.

---

## Diagram

```
           Write Request
                  ↓
             Application
                  ↓
            Update Cache
                  ↓
            Update DB
                  ↓
              Success
```

---

## Advantages

### Strong consistency

Cache and DB remain synchronized.

### Fast reads

Updated value already exists in cache.

---

## Disadvantages

### Slower writes

Every write updates both systems.

### Unused data may be cached

Even if nobody reads it later.

---

# Interview Example

### Design Instagram User Profile

When user opens profile:

```
GET /user/123
```

Use:

✅ Read-Through Cache

Reason:

* Reads are huge
* Profile changes rarely

---

When user updates profile:

```
PUT /user/123
```

Use:

✅ Write-Through Cache

Reason:

* Cache and DB stay consistent
* Users immediately see latest data

---

# One-Line Interview Answer

**Caching** stores frequently used data in memory to reduce database load and improve response time.

**Read-Through Cache**: On a cache miss, data is fetched from the database and stored in the cache before returning.

**Write-Through Cache**: Every write updates both the cache and the database, keeping them consistent at the cost of slower writes.

A common HLD pattern is:

```
Reads  → Read-Through Cache
Writes → Write-Through Cache
```

This combination appears in systems like e-commerce product pages, user profiles, social media feeds, and content platforms.


These are important HLD interview topics because interviewers often ask:

> "What happens when the cache becomes full?"
>
> "How do multiple servers share cache?"
>
> "How would you cache millions of users?"

---

# 1. Caching Strategies

A caching strategy defines **how data gets into the cache and how it stays consistent with the database.**

---

## A. Cache-Aside (Lazy Loading)

Most common strategy in interviews.

### Flow

```text
Request
   ↓
Cache
   ↓
Miss?
   ↓
Database
   ↓
Store in Cache
   ↓
Return
```

### Example

Product Page

```text
GET /product/101
```

1. Check Redis
2. Not found
3. Query DB
4. Save in Redis
5. Return data

---

### Pros

✅ Easy to implement

✅ Only frequently accessed data gets cached

✅ Saves memory

---

### Cons

❌ First request is slow

❌ Cache misses still hit DB

---

### Interview Use Cases

* Product Catalog
* User Profiles
* News Articles

---

## B. Write-Through

When data changes:

```text
Write Cache
Write Database
```

### Example

Update user name:

```text
User
 ↓
Cache Update
 ↓
DB Update
```

---

### Pros

✅ Cache always fresh

✅ Fast reads

---

### Cons

❌ Slower writes

❌ May cache unused data

---

### Use Cases

* Banking dashboards
* User settings
* Profile information

---

## C. Write-Back (Write-Behind)

Instead of writing immediately to DB:

```text
Write Cache
     ↓
Return Success
     ↓
Later write to DB
```

---

### Example

Like Counter

```text
User Likes Post
     ↓
Redis Count++
     ↓
Response Sent
     ↓
Background Worker Updates DB
```

---

### Pros

✅ Extremely fast writes

✅ Reduces DB load

---

### Cons

❌ Risk of data loss if cache crashes

❌ More complex

---

### Use Cases

* Analytics
* Counters
* View counts

---

## D. Write-Around

```text
Write → DB
Read → Cache
```

New data isn't cached immediately.

Only cache when someone reads it.

---

### Use Cases

Large datasets rarely accessed.

---

# Quick Comparison

| Strategy      | Read Speed | Write Speed | Consistency |
| ------------- | ---------- | ----------- | ----------- |
| Cache Aside   | Fast       | Normal      | Medium      |
| Write Through | Fast       | Slow        | High        |
| Write Back    | Fast       | Very Fast   | Lower       |
| Write Around  | Medium     | Fast        | Medium      |

---

# 2. Cache Eviction Policies

Cache memory is limited.

Example:

```text
Redis Memory = 10 GB
```

Eventually:

```text
Cache Full
```

Question:

> Which item should be removed?

That's cache eviction.

---

## A. LRU (Least Recently Used)

Most common interview answer.

Remove item that hasn't been accessed for the longest time.

---

### Example

Cache Size = 3

```text
A B C
```

Access:

```text
A
```

Now:

```text
B C A
```

Insert D:

```text
B removed
```

Result:

```text
C A D
```

---

### Why It Works

Assumption:

> Recently used data is likely to be used again.

Usually true.

---

### Use Cases

* Product pages
* User profiles
* Social media

---

## B. LFU (Least Frequently Used)

Remove least accessed item.

---

### Example

```text
A → 100 accesses
B → 50 accesses
C → 2 accesses
```

Need space:

```text
Remove C
```

---

### Pros

Good for highly skewed traffic.

---

### Cons

More memory overhead.

Need counters.

---

## C. FIFO (First In First Out)

Oldest item removed first.

```text
A B C
```

Insert D:

```text
Remove A
```

---

### Pros

Simple

---

### Cons

May remove popular data.

---

## D. TTL (Time To Live)

Data expires after fixed duration.

Example:

```text
Weather Data
TTL = 10 minutes
```

After 10 minutes:

```text
Automatically removed
```

---

### Use Cases

* Weather
* Stock prices
* Temporary sessions

---

# Eviction Policies Summary

| Policy | Removes               |
| ------ | --------------------- |
| LRU    | Least recently used   |
| LFU    | Least frequently used |
| FIFO   | Oldest entry          |
| TTL    | Expired entry         |

---

# 3. Distributed Caching

Interview Question:

> "One Redis server is no longer enough. What now?"

---

## Problem with Single Cache

```text
App Servers
      ↓
    Redis
```

Issues:

### Memory Limit

```text
Redis = 64 GB
Data = 500 GB
```

Not enough.

---

### Single Point of Failure

```text
Redis Down
```

Entire application slows down.

---

# Distributed Cache

Instead of one cache:

```text
Redis-1
Redis-2
Redis-3
Redis-4
```

Data spread across nodes.

---

## Sharding

Split data based on key.

Example:

```text
User1 → Redis1
User2 → Redis2
User3 → Redis3
```

---

### Hashing

```text
hash(userId) % N
```

Example:

```text
hash(101)%3 = 1
```

Store in Redis1.

---

## Consistent Hashing

Classic HLD topic.

Problem:

```text
4 Redis Nodes
Add 5th Node
```

Normal hashing causes almost all keys to move.

---

Consistent Hashing:

```text
Only small subset moves
```

Much better scalability.

Interviewers love hearing this term.

---

# Replication

For high availability:

```text
Primary Redis
      ↓
Replica Redis
      ↓
Replica Redis
```

If primary fails:

```text
Replica promoted
```

Service continues.

---

# Distributed Cache Architecture

```text
Users
  ↓
Load Balancer
  ↓
App Servers
  ↓
Redis Cluster
 ├── Node 1
 ├── Node 2
 ├── Node 3
 └── Node 4
```

---

# Interview Takeaways

### Caching Strategies

* Cache-Aside → most common
* Write-Through → strong consistency
* Write-Back → very fast writes
* Write-Around → avoid unnecessary caching

### Eviction Policies

* LRU → most common answer
* LFU → popularity-based
* FIFO → simplest
* TTL → time-based expiration

### Distributed Caching

* Multiple cache nodes
* Sharding for scale
* Replication for availability
* Consistent Hashing for easy scaling

### 30-Second HLD Answer

> "For reads, I'd use Cache-Aside with Redis. For writes, I'd use Write-Through or Write-Back depending on consistency requirements. I'd configure LRU eviction and TTLs. As traffic grows, I'd move to a distributed Redis cluster using sharding and consistent hashing, with replicas for fault tolerance."
