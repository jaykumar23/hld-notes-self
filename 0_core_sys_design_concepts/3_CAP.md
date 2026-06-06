# CAP Theorem (Very Important HLD Interview Topic)

CAP theorem is one of the most frequently discussed distributed systems concepts in HLD interviews.

It explains the trade-offs you must make when designing distributed systems.

---

# What is CAP Theorem?

CAP stands for:

```text
C = Consistency
A = Availability
P = Partition Tolerance
```

A distributed system can guarantee **at most two of these three properties simultaneously** when a network partition occurs.

---

# Before CAP: What is a Network Partition?

Suppose you have two database servers:

```text
DB1  <----->  DB2
```

Normally they communicate.

Now the network link breaks:

```text
DB1     X     DB2
```

They cannot talk to each other.

This situation is called a:

```text
Network Partition
```

This is exactly the scenario CAP theorem discusses.

---

# 1. Consistency (C)

## Meaning

Every user sees the same latest data.

After a write succeeds:

```text
User A writes:
Balance = 1000
```

Immediately:

```text
User B reads:
Balance = 1000
```

No stale data.

---

## Example

### Bank Account

Initial:

```text
Balance = ₹5000
```

User withdraws:

```text
₹1000
```

New balance:

```text
₹4000
```

Every server should return:

```text
₹4000
```

Not:

```text
Server1 → ₹4000
Server2 → ₹5000
```

That would violate consistency.

---

# 2. Availability (A)

## Meaning

Every request receives a response.

Even if some servers are failing.

The response may or may not contain the latest data.

---

## Example

Suppose:

```text
DB1
DB2
```

Connection between them breaks.

User sends request to DB2.

Availability says:

```text
DB2 must respond
```

even if data may be outdated.

---

# 3. Partition Tolerance (P)

## Meaning

System continues functioning despite network failures between nodes.

---

## Example

```text
DB1  <----X----> DB2
```

Network fails.

System should still keep working.

Modern distributed systems must tolerate partitions.

---

# Why Partition Tolerance Is Usually Mandatory

In real systems:

* Network cables fail
* Routers fail
* Data centers fail
* Cloud regions fail

Partitions are unavoidable.

Therefore:

```text
Most distributed systems must choose P
```

Then decide between:

```text
CP or AP
```

---

# The CAP Triangle

```text
       C
      / \
     /   \
    /     \
   P ----- A
```

You can have:

```text
CP
or
AP
```

But not all three during a partition.

---

# Understanding the Tradeoff

Suppose:

```text
DB1
DB2
```

Network breaks:

```text
DB1   X   DB2
```

User updates data on DB1.

Now what should DB2 do?

Two choices:

---

# Option 1: Choose Consistency (CP)

DB2 refuses reads/writes until synchronization.

```text
DB2:
"Sorry, cannot serve request."
```

Result:

✅ Consistency

❌ Availability

Because users get errors.

---

## Example

Banking Systems

Would you rather:

```text
See wrong balance?
```

or

```text
Get temporary error?
```

Banks choose:

```text
Temporary error
```

Thus:

```text
CP
```

---

# Option 2: Choose Availability (AP)

DB2 continues serving requests.

```text
DB2:
Balance = ₹5000
```

while DB1 has:

```text
₹4000
```

Result:

✅ Availability

❌ Consistency

Users may see stale data.

---

## Example

Instagram Likes

Suppose you like a post.

One user sees:

```text
100 likes
```

Another sees:

```text
101 likes
```

for a few seconds.

Not a big problem.

Instagram favors:

```text
AP
```

---

# Real World Examples

## CP Systems

Prioritize correctness.

Examples:

```text
HBase
MongoDB (majority writes)
Zookeeper
Etcd
Consul
```

Used for:

* Banking
* Payments
* Inventory
* Financial systems

---

## AP Systems

Prioritize availability.

Examples:

```text
Cassandra
DynamoDB (configurable)
Riak
DNS
```

Used for:

* Social Media
* Analytics
* Recommendation Systems

---

# Why CA Is Mostly Theoretical

People often ask:

Can we choose CA?

Answer:

Only if partitions never happen.

Example:

```text
Single database server
```

No partition possible.

Then:

```text
Consistency + Availability
```

works.

But in distributed systems:

```text
Partitions are inevitable
```

So CA is generally not realistic.

---

# Interview Scenario

### Design a Banking System

What would you choose?

Answer:

```text
CP
```

Reason:

Wrong balances are unacceptable.

Temporary unavailability is acceptable.

---

### Design Instagram Feed

Choose:

```text
AP
```

Reason:

Seeing a slightly outdated post count is acceptable.

Users should always get a response.

---

### Design Inventory Management

Suppose only 1 item left.

Two users try buying it.

Need:

```text
Strong consistency
```

Choose:

```text
CP
```

Otherwise:

```text
Overselling may happen
```

---

# Common Interview Follow-up

## Is CAP saying we cannot have all 3?

Not exactly.

The precise statement is:

> During a network partition, you must choose between Consistency and Availability.

Normally (when no partition exists), systems can often provide all three.

The tradeoff becomes important when a partition occurs.

---

# CAP Cheat Sheet

| Property            | Meaning                          |
| ------------------- | -------------------------------- |
| Consistency         | Everyone sees latest data        |
| Availability        | Every request gets a response    |
| Partition Tolerance | System survives network failures |

---

# Easy Memory Trick

### Bank

```text
Money must be correct
```

Choose:

```text
CP
```

---

### Instagram

```text
Always show something
```

Choose:

```text
AP
```

---

### Banking Example

```text
Correct balance > Uptime
```

CP

---

### Social Media Example

```text
Uptime > Perfect freshness
```

AP

---

# 60-Second Interview Answer

> CAP theorem states that in a distributed system, when a network partition occurs, we can choose either Consistency or Availability, but not both simultaneously. Consistency means all nodes see the latest data, Availability means every request receives a response, and Partition Tolerance means the system continues operating despite network failures. Since partitions are unavoidable in distributed systems, most real-world systems are designed as either CP systems (like banking and inventory systems) or AP systems (like social media feeds and DNS).
