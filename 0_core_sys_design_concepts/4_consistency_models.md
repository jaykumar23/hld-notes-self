# Consistency Models (Important HLD Interview Topic)

After CAP theorem, interviewers often ask:

> "What kind of consistency does your system provide?"

Not every system needs **strong consistency**. Different applications use different consistency models.

---

# What is Consistency?

Consistency answers:

> "After a write happens, when will other users see that write?"

Example:

```text
User A updates profile photo.
```

How quickly should User B see it?

* Immediately?
* After 1 second?
* After 1 minute?

The answer depends on the consistency model.

---

# 1. Strong Consistency

## Definition

After a successful write, every subsequent read returns the latest value.

### Example

Initial value:

```text
Balance = ₹5000
```

User A withdraws ₹1000.

```text
Balance = ₹4000
```

Immediately:

```text
User B reads
→ ₹4000
```

Always.

Never sees:

```text
₹5000
```

---

## Timeline

```text
Write Balance=4000
      |
      V
Read -> 4000
Read -> 4000
Read -> 4000
```

No stale data.

---

## Advantages

### Correct Data

Users always see latest information.

### Easy Reasoning

Application behavior is predictable.

---

## Disadvantages

### Higher Latency

Need synchronization between replicas.

### Lower Availability

Some reads may wait for replicas.

---

## Use Cases

* Banking
* Payments
* Stock Trading
* Inventory Systems

---

## Interview Example

If only one ticket remains:

```text
Ticket Count = 1
```

Two users buy simultaneously.

Need:

```text
Strong Consistency
```

Otherwise overselling occurs.

---

# 2. Eventual Consistency

Most common in large-scale systems.

---

## Definition

If no new updates occur, eventually all replicas will converge to the same value.

There may be temporary inconsistencies.

---

## Example

User updates profile picture.

```text
Replica A = New Photo
Replica B = Old Photo
Replica C = Old Photo
```

After a few seconds:

```text
Replica A = New Photo
Replica B = New Photo
Replica C = New Photo
```

Eventually everyone agrees.

---

## Timeline

```text
Write
 |
 V

Node A -> Updated

Node B -> Old Data

Node C -> Old Data

(After synchronization)

Node A -> Updated
Node B -> Updated
Node C -> Updated
```

---

## Advantages

### Very High Availability

Nodes can serve requests immediately.

### Better Scalability

No need to wait for synchronization.

### Lower Latency

Reads are fast.

---

## Disadvantages

Users may temporarily see stale data.

---

## Use Cases

* Social Media
* DNS
* Recommendation Systems
* News Feed

---

## Example

Instagram like count:

```text
100 Likes
```

One user sees:

```text
101 Likes
```

Another sees:

```text
100 Likes
```

for a few seconds.

Acceptable.

---

# Strong vs Eventual Consistency

| Strong             | Eventual            |
| ------------------ | ------------------- |
| Latest data always | Stale data possible |
| Higher latency     | Lower latency       |
| Lower availability | Higher availability |
| Banking            | Social Media        |

---

# 3. Weak Consistency

No guarantee when updates become visible.

---

## Example

User uploads profile picture.

Another user may see:

```text
Old Picture
```

for a long time.

System doesn't promise synchronization speed.

---

## Use Cases

* Analytics dashboards
* Monitoring systems
* Logging systems

---

# 4. Read-Your-Writes Consistency

Very common interview topic.

---

## Definition

A user always sees their own updates immediately.

Others may not.

---

## Example

You change profile photo.

Immediately:

```text
You see new photo
```

Friend may still see:

```text
Old photo
```

for a few seconds.

---

## Timeline

```text
User A updates profile

User A -> New Value

User B -> Old Value
```

Eventually:

```text
User B -> New Value
```

---

## Why Important?

Without it:

```text
You update profile
Refresh page
Old profile appears
```

Bad user experience.

---

## Used By

* Facebook
* Instagram
* Twitter
* Gmail

---

# 5. Monotonic Read Consistency

## Definition

Once a user sees a newer value, they never see an older value later.

---

## Bad Scenario

First request:

```text
Like Count = 100
```

Second request:

```text
Like Count = 95
```

Data moved backwards.

Users get confused.

---

## Monotonic Read Prevents This

```text
100
101
102
103
```

Never:

```text
100
102
99
```

---

## Use Cases

* Social Media Feeds
* News Feeds
* User Profiles

---

# 6. Monotonic Write Consistency

## Definition

Writes from a single user are applied in the same order they were issued.

---

## Example

User performs:

```text
Update Name
Update Address
Update Phone
```

Must be applied as:

```text
1 → 2 → 3
```

Not:

```text
3 → 1 → 2
```

---

## Use Cases

* Banking
* User Profiles
* Order Systems

---

# 7. Causal Consistency

Important advanced interview topic.

---

## Definition

Operations that are causally related must be seen in the same order by everyone.

---

## Example

User A posts:

```text
"Hello"
```

User B comments:

```text
"Nice Post"
```

Everyone should see:

```text
Hello
Nice Post
```

Not:

```text
Nice Post
Hello
```

because comment depends on post.

---

## Use Cases

* Social Networks
* Collaborative Apps
* Messaging Systems

---

# 8. Session Consistency

## Definition

Within a user session, reads are consistent.

---

## Example

You log into Amazon.

Update address:

```text
Mumbai
```

Throughout that session:

```text
Mumbai
```

is always shown.

After session ends:

Replication may continue.

---

# Database Examples

## Strong Consistency

Examples:

```text
PostgreSQL
MySQL
SQL Server
Oracle
```

Typically single-primary systems.

---

## Eventual Consistency

Examples:

```text
Cassandra
DynamoDB
Riak
DNS
```

---

## Tunable Consistency

Examples:

```text
Cassandra
DynamoDB
MongoDB
```

Can choose:

```text
Strong
Eventual
Quorum
```

depending on requirements.

---

# HLD Interview: Which Consistency Should You Choose?

### Banking System

```text
Strong Consistency
```

Wrong balance is unacceptable.

---

### Inventory System

```text
Strong Consistency
```

Avoid overselling.

---

### WhatsApp Messages

```text
Causal Consistency
```

Messages should remain ordered.

---

### Instagram Feed

```text
Eventual Consistency
```

Slight delay is acceptable.

---

### Analytics Dashboard

```text
Weak Consistency
```

Real-time accuracy isn't critical.

---

# Interview Answer (60 Seconds)

> Consistency models define how quickly writes become visible to readers in a distributed system. Strong consistency guarantees every read returns the latest data, while eventual consistency guarantees replicas converge over time and may temporarily return stale data. Systems like banking and inventory management typically require strong consistency, whereas social media feeds and DNS commonly use eventual consistency for better availability and scalability. Additional models such as read-your-writes, monotonic reads, causal consistency, and session consistency provide user-friendly guarantees without requiring full strong consistency.

## Quick Revision Table

| Consistency Model | Guarantee                 | Example          |
| ----------------- | ------------------------- | ---------------- |
| Strong            | Latest data always        | Banking          |
| Eventual          | Eventually synchronized   | Instagram        |
| Weak              | No timing guarantee       | Analytics        |
| Read-Your-Writes  | User sees own updates     | Facebook Profile |
| Monotonic Read    | Never go backward         | News Feed        |
| Monotonic Write   | Writes stay ordered       | User Updates     |
| Causal            | Cause-before-effect order | WhatsApp         |
| Session           | Consistent during session | Amazon Login     |

### Interview Tip

When designing a system, don't say:

> "I will use strong consistency everywhere."

Interviewers usually expect you to discuss the trade-off:

```text
Strong Consistency
      vs
Availability + Scalability
```

and choose the consistency model based on the business requirement.
