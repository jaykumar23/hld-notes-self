These topics are part of **Coordination & Consensus in Distributed Systems**, which are frequently asked in **System Design (HLD) interviews** because multiple servers must work together without conflicting with each other.

---

# 1. What is Coordination in Distributed Systems?

Imagine a team of 20 people working on the same Google Doc.

Questions arise:

* Who can edit?
* What if two people edit the same line?
* How do others know about changes?
* Who decides the final version?

Servers face the same problem.

**Coordination = Making multiple machines work together correctly.**

Without coordination:

* Duplicate payments may happen
* Multiple leaders may be elected
* Data may become inconsistent
* Jobs may execute twice

---

# Why Coordination is Needed

Suppose Netflix has 100 backend servers.

A task runs every midnight:

```
Generate Billing Report
```

If all 100 servers run it:

```
100 reports generated
```

Wrong.

We need:

```
Exactly one server should run it
```

Coordination helps decide:

* Who does the work?
* Who waits?
* Who becomes leader?
* Who owns a resource?

---

# What is Consensus?

Consensus means:

> Multiple servers agree on a single decision.

Example:

5 servers need to decide:

```
Who is the leader?
```

Possible votes:

```
S1 -> A
S2 -> A
S3 -> A
S4 -> B
S5 -> A
```

Result:

```
Leader = A
```

Everyone agrees.

That's consensus.

---

# Real Life Analogy

5 friends choosing a restaurant.

Options:

```
Pizza
Burger
Chinese
```

After voting:

```
Pizza wins
```

Everyone follows that decision.

Consensus = Agreement among distributed nodes.

---

# Challenges in Distributed Systems

Consensus is hard because:

### Network Delays

Server A says:

```
Leader = X
```

But message arrives late.

Some servers still think:

```
Leader = Y
```

---

### Server Crash

Leader crashes.

Need a new leader.

---

### Network Partition

Some servers can talk only among themselves.

Example:

```
Group A: S1,S2,S3

Group B: S4,S5
```

Each group might think they're correct.

Dangerous situation.

---

# Consensus Algorithms

Popular algorithms:

### Raft

Most interview-friendly.

### Paxos

Theoretical and difficult.

### Zab

Used by ZooKeeper.

### Viewstamped Replication

Less common.

For HLD interviews:

```
Say Raft
```

and you're good.

---

# Leader Election

One of the most common coordination mechanisms.

---

## What is Leader Election?

Selecting one node as:

```
Leader
```

and others as:

```
Followers
```

Leader handles:

* Writes
* Coordination
* Scheduling
* Metadata updates

Followers replicate state.

---

## Example

Suppose:

```
Server A
Server B
Server C
```

Initially:

```
Leader = A
```

Clients send requests to A.

---

### Leader Failure

A crashes.

Now system needs:

```
New leader
```

Election starts.

---

### Voting

B and C vote.

Suppose:

```
B gets majority
```

Then:

```
Leader = B
```

System continues.

---

# Why Leaders?

Without leader:

Two servers may update same data simultaneously.

Example:

```
Account Balance = 100
```

Server1:

```
+50
```

Server2:

```
-30
```

Results may conflict.

Leader serializes updates.

---

# Raft Leader Election (Interview Version)

States:

```
Follower
Candidate
Leader
```

---

### Step 1

All nodes start as:

```
Followers
```

---

### Step 2

No heartbeat received.

Timeout occurs.

Example:

```
Server B timeout
```

B becomes:

```
Candidate
```

---

### Step 3

B asks for votes.

```
Vote for me?
```

---

### Step 4

Majority votes yes.

```
B becomes Leader
```

---

### Step 5

Leader sends heartbeats.

```
I am alive
```

Followers reset timers.

No more elections.

---

# Interview One-Liner

> Leader Election chooses one node to coordinate the cluster. If the leader fails, nodes vote and elect a new leader using algorithms like Raft.

---

# Gossip Protocol

A very common HLD topic.

Used in:

* Cassandra
* DynamoDB
* Redis Cluster
* Consul
* RingPop

---

# What is Gossip Protocol?

Information spreads similarly to human gossip.

---

## Human Example

You tell:

```
2 friends
```

Each tells:

```
2 more friends
```

Soon:

```
Everyone knows
```

---

## Server Example

Node A learns:

```
Server D failed
```

A tells:

```
B and C
```

B tells:

```
E and F
```

C tells:

```
G and H
```

Soon entire cluster knows.

---

# Visualization

```
      A
    /   \
   B     C
  / \   / \
 D  E  F  G
```

Information propagates exponentially.

---

# Why Gossip?

Without gossip:

```
A informs every node
```

Cost:

```
O(N)
```

Large clusters become expensive.

---

With gossip:

```
O(log N)
```

Very scalable.

---

# What Information is Shared?

* Health status
* Membership
* Node failures
* Load information
* Metadata

---

# Interview Example

Suppose:

```
1000 servers
```

One server crashes.

Instead of informing:

```
999 servers
```

Directly,

it gossips to a few nodes.

Information eventually reaches everyone.

---

# Advantages

### Highly Scalable

Works for huge clusters.

### Fault Tolerant

No central server.

### Simple

Easy to implement.

---

# Drawbacks

### Eventual Consistency

Some nodes learn later.

---

### Temporary Mismatch

For a short time:

```
Node A knows
Node B doesn't know
```

---

# Interview One-Liner

> Gossip Protocol spreads cluster information probabilistically by having nodes periodically exchange state with a few random peers.

---

# Distributed Locking

One of the most important interview topics.

---

# Problem

Suppose:

```
Server A
Server B
Server C
```

want to update:

```
Order #123
```

at the same time.

Without locking:

```
Duplicate processing
```

can occur.

---

# What is Distributed Lock?

A lock shared across multiple machines.

Only one server can hold it.

---

# Example

Order Processing:

```
Lock(Order123)
```

Server A acquires lock.

```
A -> Processing
```

B and C wait.

After completion:

```
Unlock(Order123)
```

Now another server may proceed.

---

# Visualization

```
         Order123

A ---> LOCK ACQUIRED

B ---> WAIT

C ---> WAIT
```

---

# Where Are Locks Stored?

Usually in:

* ZooKeeper
* etcd
* Redis

---

# Redis Distributed Lock

Popular interview answer.

Using:

```
SET key value NX EX 30
```

Meaning:

```
Create lock only if absent
Expire after 30 seconds
```

Example:

```
SET order123 locked NX EX 30
```

Success:

```
Lock acquired
```

Failure:

```
Someone else owns lock
```

---

# Why Expiration?

Suppose server crashes.

Without expiration:

```
Lock never released
```

Deadlock.

Expiry prevents this.

---

# Common Use Cases

### Payment Processing

Prevent duplicate payments.

### Inventory Management

Prevent overselling.

### Cron Jobs

Only one server executes job.

### Leader Election

Leader ownership can be a lock.

---

# Interview One-Liner

> Distributed Locking ensures that only one node can access a shared resource at a time, typically implemented using Redis, ZooKeeper, or etcd.

---

# Quick HLD Interview Revision Table

| Concept             | Simple Meaning                  | Purpose                |
| ------------------- | ------------------------------- | ---------------------- |
| Coordination        | Servers work together correctly | Avoid conflicts        |
| Consensus           | All nodes agree on one decision | Consistency            |
| Leader Election     | Pick one coordinator node       | Manage cluster         |
| Gossip Protocol     | Spread information gradually    | Cluster communication  |
| Distributed Locking | One server accesses resource    | Prevent duplicate work |

# 30-Second Interview Answer

> In distributed systems, coordination ensures multiple servers work together correctly. Consensus algorithms like Raft help nodes agree on decisions such as leader election. Leader election selects a single coordinator node to handle writes and cluster management. Gossip Protocol is used to efficiently spread cluster state information across nodes. Distributed locking ensures only one node accesses a shared resource at a time, preventing duplicate processing and maintaining consistency.
