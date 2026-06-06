# Load Balancers (Very Important HLD Interview Topic)

Load balancing is one of the most common topics in System Design interviews because almost every large-scale system uses it.

Examples:

* Google Search
* YouTube
* Netflix
* Amazon
* Facebook

All use load balancers to distribute traffic across many servers.

---

# 1. What is a Load Balancer?

A Load Balancer sits between clients and servers.

```text
Users
   |
   v
+----------------+
| Load Balancer  |
+----------------+
   |    |    |
   v    v    v
Server1 Server2 Server3
```

Instead of users directly hitting servers:

```text
User -> Server1
User -> Server2
User -> Server3
```

they hit:

```text
User -> Load Balancer -> Servers
```

The LB decides:

* Which server receives request
* How traffic is distributed
* Which servers are healthy

---

# Why do we need Load Balancers?

Imagine:

```text
Server1 = 1000 requests/sec capacity
```

Traffic becomes:

```text
10,000 requests/sec
```

One server crashes.

Instead:

```text
10 servers × 1000 RPS
```

Traffic gets distributed.

Benefits:

### Scalability

Add more servers.

```text
Before:
1 Server

After:
10 Servers
```

---

### High Availability

If one server dies:

```text
Server1 ❌
Server2 ✅
Server3 ✅
```

LB routes traffic only to healthy servers.

---

### Fault Tolerance

Users don't notice server failures.

---

### Better Resource Utilization

Instead of:

```text
Server1 = 90%
Server2 = 10%
```

LB can make:

```text
Server1 = 50%
Server2 = 50%
```

---

# Layer 4 vs Layer 7 Load Balancers

---

# Layer 4 Load Balancer (Transport Layer)

Works using:

```text
IP Address
TCP Port
```

Example:

```text
Client -> LB
LB -> Server
```

LB doesn't inspect HTTP content.

It only sees:

```text
Source IP
Destination IP
Port
```

Examples:

* AWS NLB
* HAProxy (TCP mode)

---

### Pros

Very fast

```text
Millions of requests/sec
```

Low latency.

---

### Cons

Cannot route based on:

```text
/api
/images
/users
```

---

# Layer 7 Load Balancer (Application Layer)

Understands HTTP.

Can inspect:

```http
GET /api/users
Host: app.com
```

---

Example:

```text
/api/*     -> Backend Cluster
/images/*  -> Image Cluster
/videos/*  -> Video Cluster
```

```text
Client
   |
   v
Layer 7 LB
   |
   +--> API Servers
   +--> Image Servers
   +--> Video Servers
```

---

### Examples

* NGINX
* Envoy
* AWS ALB

---

### Pros

Smart routing

Path-based routing:

```text
/api -> Cluster A
```

Host-based routing:

```text
api.company.com
```

Header-based routing.

---

### Cons

More CPU overhead.

---

# Health Checks

LB continuously checks servers.

```text
LB --> Server1
      /health
```

Response:

```text
200 OK
```

Healthy.

---

If:

```text
500 Error
```

or timeout:

```text
Server1 ❌
```

Traffic stops.

---

# Sticky Sessions (Session Affinity)

Normally:

```text
Req1 -> Server1
Req2 -> Server2
Req3 -> Server3
```

But some applications store session in memory.

Example:

```text
Shopping Cart
```

Need same server.

```text
UserA
   |
   +--> Server2
   +--> Server2
   +--> Server2
```

---

Methods:

### Cookie Based

```text
session_id=abc
```

---

### IP Hash

```text
hash(IP) -> Server
```

---

Problem:

Server failure loses session.

Modern systems use:

```text
Redis
Database
```

instead of sticky sessions.

---

# Load Balancing Algorithms

The LB needs a strategy to select a server.

---

# 1. Round Robin

Most common.

```text
Request1 -> Server1
Request2 -> Server2
Request3 -> Server3
Request4 -> Server1
```

Cycle repeats.

---

### Pros

Simple.

---

### Cons

Assumes all servers equal.

---

# 2. Weighted Round Robin

Servers may have different capacities.

Example:

```text
Server1 = Weight 5
Server2 = Weight 2
Server3 = Weight 1
```

Traffic:

```text
S1 S1 S1 S1 S1
S2 S2
S3
```

---

Used when:

```text
32-core machine
8-core machine
4-core machine
```

exist together.

---

# 3. Least Connections

Choose server with fewest active connections.

```text
Server1 = 100 connections
Server2 = 30 connections
Server3 = 10 connections
```

Choose:

```text
Server3
```

---

Useful when request duration varies.

---

Example:

```text
Video upload
```

Some requests may last minutes.

---

# 4. Weighted Least Connections

Combines:

```text
Server Capacity
+
Active Connections
```

Common in production.

---

# 5. IP Hash

```text
hash(client_ip)
```

Example:

```text
192.168.1.10 -> Server2
```

Always same server.

Useful for:

```text
Sticky Sessions
```

---

# 6. Random

Pick random server.

Surprisingly effective at scale.

---

# 7. Consistent Hashing

Popular in distributed systems.

Used in:

* Redis Cluster
* Cassandra
* DynamoDB

Idea:

```text
hash(user_id)
```

Maps request to server.

Adding/removing servers causes minimal remapping.

---

# DNS Load Balancing

Instead of a load balancer machine, DNS returns multiple IPs.

---

Normal DNS:

```text
api.company.com
       |
       v
    1 IP
```

---

DNS LB:

```text
api.company.com
       |
       v
IP1
IP2
IP3
```

DNS rotates responses.

```text
User1 -> IP1
User2 -> IP2
User3 -> IP3
```

---

### Advantages

Cheap.

Global.

Simple.

---

### Problems

DNS caching.

```text
TTL = 1 hour
```

User may keep using dead server.

No real-time health checks.

Less control.

---

# Anycast Routing

One of the coolest load-balancing techniques.

Multiple servers advertise the same IP.

```text
Global IP:
8.8.8.8
```

Exists in many locations.

```text
Mumbai
Singapore
London
New York
```

All announce:

```text
8.8.8.8
```

---

Internet routing sends user to nearest location.

```text
Mumbai User
   |
   v
Mumbai Data Center
```

```text
London User
   |
   v
London Data Center
```

---

Used by:

* Cloudflare
* Google DNS (8.8.8.8)
* AWS Route53
* Fastly

---

Benefits

### Low Latency

Traffic reaches nearest POP.

---

### DDoS Protection

Attack distributed across many data centers.

---

### Global Scaling

No single entry point.

---

# Global vs Regional Load Balancers

This is frequently asked in HLD.

---

# Regional Load Balancer

Works inside one region.

Example:

```text
Mumbai Region
```

```text
LB
 |
 +--> Server1
 +--> Server2
 +--> Server3
```

Traffic already reached Mumbai.

LB distributes locally.

---

Examples:

* AWS ALB
* AWS NLB
* NGINX

---

# Global Load Balancer

Works across regions.

```text
User
 |
 v
Global LB
 |
 +--> Mumbai
 +--> Singapore
 +--> London
 +--> Virginia
```

Chooses best region.

---

Decision factors:

### Geography

```text
India User
→ Mumbai
```

---

### Latency

```text
Fastest Region
```

---

### Capacity

```text
Mumbai overloaded
→ Singapore
```

---

### Disaster Recovery

```text
Mumbai Down
→ Singapore
```

Automatically failover.

---

Examples:

* AWS Route53
* Google Global LB
* Cloudflare
* Akamai

---

# Real Production Architecture

```text
                 Users
                    |
                    v
        +----------------------+
        | Global Load Balancer |
        +----------------------+
            /            \
           /              \
          v                v

    Mumbai Region     Singapore Region

      Regional LB      Regional LB
          |                |
      +---+---+        +---+---+
      |       |        |       |
      v       v        v       v

   App1    App2     App1    App2
```

Flow:

```text
User
 → Global LB
 → Best Region
 → Regional LB
 → Application Server
```

---

# Interview One-Line Summary

### Round Robin

```text
Send requests sequentially across servers.
```

### Least Connections

```text
Send request to server with fewest active connections.
```

### Consistent Hashing

```text
Minimal remapping when servers are added/removed.
```

### DNS Load Balancing

```text
DNS returns multiple server IPs.
```

### Anycast

```text
Same IP advertised globally; network routes user to nearest location.
```

### Regional LB

```text
Balances traffic inside one region.
```

### Global LB

```text
Chooses the best region across the world.
```

### HLD Rule

For internet-scale systems (Netflix, YouTube, Amazon):

```text
Users
   ↓
Global Load Balancer
   ↓
Regional Load Balancer
   ↓
Application Servers
```

This is the architecture interviewers typically expect in large-scale HLD designs.
