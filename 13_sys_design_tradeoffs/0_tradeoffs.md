These are some of the most frequently asked HLD interview tradeoffs. The interviewer usually wants to know **when to choose one over the other and why**, not just definitions.

---

# 1. Vertical Scaling vs Horizontal Scaling

## Vertical Scaling (Scale Up)

Increase resources of a single machine.

Example:

* 8GB RAM → 64GB RAM
* 4 CPU → 32 CPU

```
Before:
[ Server ]

After:
[ Bigger Server ]
```

### Advantages

✅ Easy to implement

✅ No code changes

✅ No distributed system complexity

### Disadvantages

❌ Hardware limit exists

❌ Single point of failure

❌ Expensive at higher scale

### Use Cases

* Small applications
* Early-stage startups
* Internal tools

---

## Horizontal Scaling (Scale Out)

Add more machines.

```
[ Server 1 ]
[ Server 2 ]
[ Server 3 ]
[ Server 4 ]
```

### Advantages

✅ Virtually unlimited growth

✅ High availability

✅ Fault tolerant

### Disadvantages

❌ Load balancing needed

❌ Data partitioning required

❌ Distributed system complexity

### Use Cases

* Facebook
* Netflix
* Amazon
* YouTube

---

## Interview Answer

> Vertical scaling is simpler but has hardware limits and single point of failure. Horizontal scaling is more complex but provides better availability and almost unlimited scalability, making it preferred for large-scale distributed systems.

---

# 2. Concurrency vs Parallelism

Many candidates confuse these.

---

## Concurrency

Multiple tasks make progress together.

One CPU can handle many tasks by switching rapidly.

```
Task A
Task B
Task C

CPU:
A -> B -> C -> A -> B
```

### Example

Node.js server handling thousands of requests.

### Goal

Improve responsiveness.

---

## Parallelism

Multiple tasks execute at exactly the same time.

```
Core 1 -> Task A
Core 2 -> Task B
Core 3 -> Task C
```

### Example

Image processing

Video rendering

ML training

### Goal

Improve speed.

---

## Interview Answer

> Concurrency is about dealing with many tasks at once, while parallelism is about executing multiple tasks simultaneously. Concurrency improves responsiveness, whereas parallelism improves throughput.

---

# 3. Long Polling vs WebSockets

Used for real-time communication.

---

## Long Polling

Client repeatedly asks server for updates.

```
Client ---> Request
Server ---> Response

Client ---> Request
Server ---> Response
```

### Pros

✅ Easy to implement

✅ Works with existing HTTP infrastructure

### Cons

❌ Repeated connections

❌ High latency

❌ More network overhead

### Examples

* Notification systems
* Older chat systems

---

## WebSockets

Persistent connection between client and server.

```
Client <========> Server
```

Both can send data anytime.

### Pros

✅ Real-time

✅ Low latency

✅ Efficient

### Cons

❌ More infrastructure complexity

❌ Connection management

### Examples

* WhatsApp
* Slack
* Trading apps
* Multiplayer games

---

## Interview Answer

> Long polling is simpler but creates repeated HTTP requests and higher latency. WebSockets maintain a persistent bidirectional connection and are preferred for highly interactive real-time applications.

---

# 4. Stateful vs Stateless Architecture

---

## Stateful

Server remembers user state.

```
User -> Server A
Session stored on Server A
```

Next request must go to same server.

### Advantages

✅ Simple session management

### Disadvantages

❌ Hard to scale

❌ Session affinity required

❌ Failure causes session loss

---

## Stateless

Every request contains all required information.

```
JWT Token
User ID
Auth Info
```

Any server can process request.

### Advantages

✅ Easy scaling

✅ Load balancing friendly

✅ Fault tolerant

### Disadvantages

❌ More data sent per request

### Examples

REST APIs

Microservices

---

## Interview Answer

> Stateless systems are preferred for large-scale distributed architectures because any server can serve any request, making scaling and failover easier.

---

# 5. Strong Consistency vs Eventual Consistency

One of the most important HLD topics.

---

## Strong Consistency

After write succeeds:

```
Write X = 10

Immediately

All users see X = 10
```

### Advantages

✅ Always latest data

### Disadvantages

❌ Higher latency

❌ Lower availability

### Examples

* Banking
* Payment systems
* Stock trading

---

## Eventual Consistency

Updates propagate gradually.

```
Write X = 10

Server A -> 10
Server B -> 9

Few seconds later

Server B -> 10
```

### Advantages

✅ High availability

✅ Fast reads

✅ Better scalability

### Disadvantages

❌ Stale data possible

### Examples

* Social media likes
* Product views
* Analytics

---

## Interview Answer

> Strong consistency guarantees fresh data but increases latency. Eventual consistency improves scalability and availability but may temporarily return stale data.

---

# 6. Push vs Pull Architecture

---

## Pull

Consumer requests data.

```
Consumer ---> Producer
```

### Examples

* Browser refresh
* Polling APIs

### Advantages

✅ Simpler

### Disadvantages

❌ Unnecessary requests

❌ Higher latency

---

## Push

Producer sends updates automatically.

```
Producer ---> Consumer
```

### Examples

* Notifications
* WebSockets
* Kafka consumers

### Advantages

✅ Real-time

✅ Low latency

### Disadvantages

❌ More complex

---

## Interview Answer

> Pull is simpler but can waste resources due to frequent polling. Push provides near real-time updates but requires maintaining delivery mechanisms.

---

# 7. Cost vs Performance Tradeoff

This comes up in every system design interview.

---

## High Performance

To reduce latency:

* More servers
* More replicas
* Better hardware
* CDN
* Caching

### Result

```
Latency ↓
Cost ↑
```

---

## Low Cost

To save money:

* Fewer servers
* Smaller caches
* Lower replication

### Result

```
Cost ↓
Latency ↑
```

---

## Example

Global CDN:

```
Without CDN:
100 ms latency

With CDN:
20 ms latency
```

But CDN increases infrastructure cost.

---

## Interview Answer

> System design is often about balancing performance against infrastructure cost. Features like caching, replication, and CDNs improve performance but increase operational expenses.

---

# 8. Latency vs Consistency Tradeoff

A direct consequence of the **CAP theorem**.

---

## Low Latency

Return data immediately.

```
User -> Nearest Replica
```

### Problem

Replica may be stale.

---

## High Consistency

Wait until all replicas synchronize.

```
Write
↓
Replica Sync
↓
Response
```

### Problem

Slower response.

---

## Example

Instagram Like Count

Low latency is more important.

```
Like count:
101
102
103
```

Being off by 1-2 likes is acceptable.

---

## Example

Bank Account Balance

Consistency is more important.

```
Balance:
₹1000
```

Showing ₹900 or ₹1100 is unacceptable.

---

## Interview Answer

> To achieve strong consistency, systems often wait for replica synchronization, increasing latency. Systems that prioritize low latency may serve slightly stale data. The choice depends on business requirements.

---

# One-Line Interview Cheat Sheet

| Tradeoff             | Preferred When               |
| -------------------- | ---------------------------- |
| Vertical Scaling     | Small systems                |
| Horizontal Scaling   | Large distributed systems    |
| Concurrency          | Many simultaneous users      |
| Parallelism          | Faster computation           |
| Long Polling         | Simple real-time features    |
| WebSockets           | True real-time communication |
| Stateful             | Small/simple applications    |
| Stateless            | Scalable distributed systems |
| Strong Consistency   | Banking, payments            |
| Eventual Consistency | Social media, analytics      |
| Push                 | Real-time updates            |
| Pull                 | Simpler integrations         |
| Performance          | User experience critical     |
| Cost                 | Budget constrained           |
| Low Latency          | User-facing systems          |
| Strong Consistency   | Financial/critical systems   |

For HLD interviews, a strong answer usually follows this pattern:

**"The choice depends on scale, business requirements, latency requirements, consistency requirements, and cost constraints."**

That single sentence fits almost every architecture tradeoff discussion.
