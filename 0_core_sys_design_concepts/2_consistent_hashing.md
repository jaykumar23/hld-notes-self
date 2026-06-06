## Consistent Hashing (Simple HLD Interview Explanation)

Consistent Hashing is a technique used to **distribute data across multiple servers** so that when servers are added or removed, **only a small amount of data needs to move**.

Used in:

* Distributed Caches (Redis Cluster, Memcached)
* Distributed Databases (Cassandra, DynamoDB)
* Load Balancers
* CDNs

---

# Problem with Normal Hashing

Suppose you have 4 servers:

```
S1 S2 S3 S4
```

You store data using:

```cpp
server = hash(key) % 4
```

Example:

```
User1 -> S1
User2 -> S3
User3 -> S2
```

### What if a new server is added?

Now:

```cpp
server = hash(key) % 5
```

Almost every key gets a different result.

```
User1 -> S4
User2 -> S1
User3 -> S5
```

### Problem

Nearly **100% of data must be redistributed**.

This is expensive for:

* Databases
* Caches
* Large systems

---

# Consistent Hashing Solution

Instead of using `% N`, place servers and keys on a **hash ring**.

Imagine a circle:

```
0 -----------------> MAX_HASH
 \                 /
  \               /
   ---------------
```

Hash values wrap around.

---

## Step 1: Place Servers on Ring

Hash each server:

```
hash(S1)=100
hash(S2)=300
hash(S3)=500
hash(S4)=800
```

Ring:

```
         S1(100)

   S4(800)       S2(300)

         S3(500)
```

---

## Step 2: Place Keys

Example:

```
hash(UserA)=250
```

Move clockwise until finding first server.

```
250 -> S2(300)
```

Example:

```
hash(UserB)=450
```

```
450 -> S3(500)
```

Example:

```
hash(UserC)=850
```

Wrap around:

```
850 -> S1(100)
```

---

# Server Addition Example

Current:

```
S1(100)
S2(300)
S3(500)
S4(800)
```

Add:

```
S5(400)
```

New ring:

```
S1(100)
S2(300)
S5(400)
S3(500)
S4(800)
```

Only keys between:

```
300 → 400
```

move from:

```
S3 → S5
```

Everything else remains unchanged.

### Huge Benefit

Only a small portion of keys move.

---

# Server Removal Example

Remove:

```
S3(500)
```

Only keys owned by S3 move.

```
S3 keys → S4
```

Other servers unaffected.

---

# Why Is It Good?

### Normal Hashing

```
Add 1 server
=> Almost all keys move
```

### Consistent Hashing

```
Add 1 server
=> Only neighboring keys move
```

This property makes systems highly scalable.

---

# Virtual Nodes (Very Important Interview Topic)

### Problem Without Virtual Nodes

Suppose only 3 servers:

```
S1(100)
S2(200)
S3(900)
```

Ring:

```
100 ----- 200 ------------------------- 900
```

S3 owns a huge range.

Result:

```
S3 gets most traffic
```

Load imbalance.

---

## Solution: Virtual Nodes (VNodes)

Instead of placing each server once, place it many times.

Example:

```
S1-1(100)
S1-2(400)
S1-3(700)

S2-1(200)
S2-2(500)
S2-3(800)

S3-1(300)
S3-2(600)
S3-3(900)
```

Ring:

```
100 200 300 400 500 600 700 800 900
```

Now data is spread much more evenly.

---

## Real Mapping

Physical server:

```
Server A
```

Virtual nodes:

```
A1
A2
A3
A4
A5
...
```

All map back to the same physical machine.

---

# Why Virtual Nodes Are Important

### 1. Better Load Balancing

Without VNodes:

```
Server A = 70%
Server B = 20%
Server C = 10%
```

With VNodes:

```
A = 34%
B = 33%
C = 33%
```

Approximately equal.

---

### 2. Easier Scaling

Add a new server:

```
Server D
```

Only some virtual nodes are redistributed.

Less data movement.

---

### 3. Handle Unequal Servers

Suppose:

```
A = 16 CPU
B = 8 CPU
```

Assign:

```
A = 200 virtual nodes
B = 100 virtual nodes
```

More powerful server gets more traffic.

---

# Pros of Consistent Hashing

### 1. Minimal Data Movement

Only affected keys move.

### 2. Easy Horizontal Scaling

Add/remove servers easily.

### 3. High Availability

Node failure impacts only a subset of data.

### 4. Load Distribution

Especially with virtual nodes.

### 5. Widely Used

Used in:

* Cassandra
* DynamoDB
* Redis Cluster concepts
* Akamai CDN
* Memcached

---

# Cons of Consistent Hashing

### 1. More Complex

Harder than simple modulo hashing.

### 2. Uneven Distribution

Without virtual nodes, some servers may get much more traffic.

### 3. Metadata Management

Need to maintain ring information.

### 4. Lookup Overhead

Need to find the next clockwise server on the ring.

Typically solved using:

* Sorted Map
* Balanced BST
* Skip List

---

# Interview 30-Second Answer

> Consistent hashing distributes keys and servers on a hash ring. A key is assigned to the first server found in the clockwise direction. Unlike `hash(key) % N`, adding or removing a server causes only a small subset of keys to move. To avoid uneven load distribution, systems use virtual nodes, where each physical server is represented by multiple positions on the ring. This provides better load balancing, scalability, and fault tolerance in distributed systems.
