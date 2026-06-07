These are **advanced distributed system patterns** that frequently appear in **Senior SDE / HLD interviews**, especially when designing large-scale systems using microservices.

A good interview answer should cover:

1. What it is
2. Why it exists
3. How it works
4. Real-world example
5. Pros & Cons
6. When to use

---

# 1. Service Discovery

## Problem

Suppose you have 100 microservices.

```
User Service
Order Service
Payment Service
Inventory Service
```

Each service runs on multiple servers.

```
Order Service

10.0.1.2
10.0.1.3
10.0.1.4
```

Servers may crash and new servers may start.

How does Payment Service know where Order Service is running?

Hardcoding IPs won't work.

---

## Solution

Use Service Discovery.

A central registry keeps track of all service locations.

```
          Service Registry
                |
        -----------------
        |       |       |
      User    Order   Payment
```

Examples:

* Eureka
* Consul
* ZooKeeper
* Kubernetes Service Discovery

---

## How It Works

### Registration

Service starts:

```
Order Service
IP: 10.0.1.2
```

Registers itself.

```
Registry:
Order -> 10.0.1.2
```

---

### Lookup

Payment service asks:

```
Where is Order Service?
```

Registry returns:

```
10.0.1.2
10.0.1.3
10.0.1.4
```

Payment chooses one.

---

## Interview Example

Uber:

```
Ride Service
Payment Service
Driver Service
```

Services constantly scale.

Service discovery helps locate them.

---

## Pros

✅ Dynamic scaling

✅ No hardcoded IPs

✅ Supports load balancing

---

## Cons

❌ Registry becomes critical

❌ Extra network call

---

## One-Line Interview Answer

> Service Discovery is a mechanism that allows microservices to dynamically find each other without hardcoded network addresses.

---

# 2. API Gateway Pattern

## Problem

Without gateway:

```
Client
  |
  +--> User Service
  |
  +--> Order Service
  |
  +--> Payment Service
  |
  +--> Inventory Service
```

Client must know every service.

Very messy.

---

## Solution

Introduce API Gateway.

```
             API Gateway
                  |
      ------------------------
      |      |      |        |
    User   Order Payment Inventory
```

Single entry point.

---

## Example

Mobile app:

```
GET /userProfile
```

Gateway internally calls:

```
User Service
Order Service
Reward Service
```

Combines responses.

Returns:

```
User Profile Page
```

---

## Responsibilities

### Authentication

```
JWT Validation
```

### Rate Limiting

```
100 requests/minute
```

### Routing

```
/users -> User Service
/orders -> Order Service
```

### Aggregation

Multiple services → single response.

---

## Examples

* Kong
* NGINX
* Envoy
* AWS API Gateway

---

## Pros

✅ Single entry point

✅ Security centralized

✅ Easy monitoring

---

## Cons

❌ Single point of failure

❌ Can become bottleneck

---

## Interview Answer

> API Gateway acts as a single entry point for clients and handles routing, authentication, rate limiting, and request aggregation.

---

# 3. Backend For Frontend (BFF)

## Problem

Mobile and Web need different data.

Example:

### Mobile

```
Name
Profile Pic
```

### Web

```
Name
Profile Pic
Orders
Recommendations
Analytics
```

Single backend returns too much data.

---

## Solution

Dedicated backend per frontend.

```
            Services
                |
        ----------------
        |              |
      BFF-Web      BFF-Mobile
        |              |
      Web App      Mobile App
```

---

## Example

Netflix

Web UI needs:

```
Large banners
Recommendations
```

Mobile needs:

```
Compact response
```

Different BFFs optimize responses.

---

## Pros

✅ Frontend-specific optimization

✅ Faster APIs

✅ Smaller payloads

---

## Cons

❌ More services

❌ Duplicate logic possible

---

## Interview Answer

> BFF provides frontend-specific APIs tailored for a particular client such as web, mobile, or TV applications.

---

# 4. Sidecar Pattern

## Problem

Every service needs:

```
Logging
Monitoring
Security
Retries
```

Duplicating code everywhere is bad.

---

## Solution

Attach helper container.

```
+----------------+
| App Container  |
+----------------+
| Sidecar        |
+----------------+
```

Same pod.

---

## Kubernetes Example

```
App
|
Sidecar:
   Logging Agent
```

Logs collected automatically.

---

## Examples

Sidecars:

```
Envoy
Fluentd
Log Collectors
```

---

## Pros

✅ Reusable

✅ Separation of concerns

✅ Consistent behavior

---

## Cons

❌ More resources

❌ Operational complexity

---

## Interview Answer

> A Sidecar is a helper service deployed alongside an application to provide cross-cutting concerns such as logging, monitoring, or networking.

---

# 5. Circuit Breaker Pattern

## Problem

Payment Service depends on Bank Service.

```
Payment -> Bank
```

Bank becomes slow.

Requests pile up.

System collapses.

---

## Solution

Circuit Breaker.

Like an electrical fuse.

---

### Closed State

Normal.

```
Payment -> Bank
```

---

### Open State

Failures exceed threshold.

```
Payment X-> Bank
```

Immediately reject requests.

---

### Half Open

After cooldown:

```
Try few requests
```

If successful:

```
Close circuit
```

---

## Example

Netflix pioneered this.

---

## Pros

✅ Prevents cascading failures

✅ Faster failure response

---

## Cons

❌ Tuning thresholds

---

## Interview Answer

> Circuit Breaker prevents repeated calls to a failing service and protects the system from cascading failures.

---

# 6. Bulkhead Pattern

## Problem

Imagine a ship.

If one compartment floods:

```
Whole ship sinks
```

unless compartments are isolated.

---

## Same Idea

One service consumes all resources.

```
Thread Pool
Memory
Connections
```

Other services become unavailable.

---

## Solution

Isolation.

```
Payment Pool
Search Pool
Order Pool
```

Separate resources.

---

## Example

Amazon.

Search traffic spike should not affect checkout.

---

## Pros

✅ Fault isolation

✅ Better reliability

---

## Cons

❌ Resource wastage

---

## Interview Answer

> Bulkhead Pattern isolates resources so that failure in one component does not affect the entire system.

---

# 7. Strangler Fig Pattern

## Problem

Large monolith.

```
500k LOC
```

Cannot rewrite everything.

---

## Solution

Gradually replace pieces.

Named after the strangler fig tree.

---

### Step 1

```
Client
   |
 Monolith
```

---

### Step 2

Extract User Service.

```
Client
  |
Proxy
 | \
User Monolith
```

---

### Step 3

Extract Order Service.

---

### Final

```
Microservices
```

Monolith removed.

---

## Example

Many banks migrate this way.

---

## Pros

✅ Low risk migration

✅ Incremental

---

## Cons

❌ Long transition

❌ Hybrid architecture

---

## Interview Answer

> Strangler Fig Pattern gradually replaces a monolithic application with microservices without a full rewrite.

---

# 8. Service Mesh

## Problem

Each service needs:

```
Retries
TLS
Load Balancing
Observability
Traffic Control
```

Implementing in every service is painful.

---

## Solution

Move networking to infrastructure layer.

```
Service A <-> Envoy
                |
             Envoy <-> Service B
```

---

## Architecture

```
+---------+      +---------+
|Service A|      |Service B|
+---------+      +---------+
    |                |
  Envoy            Envoy
     \             /
      \           /
      Control Plane
```

---

## Capabilities

### Traffic Routing

```
90% -> v1
10% -> v2
```

---

### Retries

Automatic.

---

### TLS

Automatic encryption.

---

### Monitoring

Automatic metrics.

---

## Examples

* Istio
* Linkerd
* Consul Connect

---

## Pros

✅ Centralized networking

✅ Security

✅ Observability

---

## Cons

❌ Complexity

❌ Resource overhead

---

## Interview Answer

> Service Mesh is an infrastructure layer that manages service-to-service communication, security, retries, observability, and traffic routing.

---

# 9. Distributed Configuration

## Problem

100 services need:

```
DB URL
API Keys
Feature Flags
Timeouts
```

Hardcoding is bad.

---

## Solution

Centralized configuration.

```
Config Server
      |
-------------------
|   |   |   |   |
S1 S2 S3 S4 S5
```

---

## Example

Change:

```
timeout=5 sec
```

to

```
timeout=10 sec
```

Update once.

All services receive it.

---

## Common Tools

* Spring Cloud Config
* Consul
* ZooKeeper
* Kubernetes ConfigMaps

---

## Real Example

Feature Flag:

```
new_payment_flow=true
```

Enable for all services instantly.

---

## Pros

✅ Centralized management

✅ Dynamic updates

✅ Easier operations

---

## Cons

❌ Config server dependency

❌ Security concerns

---

## Interview Answer

> Distributed Configuration centralizes application settings so multiple services can share and update configuration consistently without redeployment.

---

# Quick Revision Table

| Pattern                   | Main Purpose                            |
| ------------------------- | --------------------------------------- |
| Service Discovery         | Find service locations dynamically      |
| API Gateway               | Single entry point for clients          |
| BFF                       | Separate backend per frontend           |
| Sidecar                   | Add common functionality beside service |
| Circuit Breaker           | Stop calling failing service            |
| Bulkhead                  | Isolate failures/resources              |
| Strangler Fig             | Gradual monolith migration              |
| Service Mesh              | Manage service-to-service communication |
| Distributed Configuration | Centralized config management           |

### Easy Memory Trick

**"Find → Enter → Customize → Assist → Protect → Isolate → Migrate → Connect → Configure"**

```
Service Discovery     → Find services
API Gateway           → Enter system
BFF                   → Customize APIs
Sidecar               → Assist services
Circuit Breaker       → Protect system
Bulkhead              → Isolate failures
Strangler Fig         → Migrate monolith
Service Mesh          → Connect services
Distributed Config    → Configure everything
```

These 9 patterns together form the core of most modern microservice architectures used by companies like Netflix, Uber, Amazon, and Google.
