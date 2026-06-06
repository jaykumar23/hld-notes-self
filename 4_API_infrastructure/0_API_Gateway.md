# API Gateway (HLD Interview Guide)

API Gateway is one of the most common components in modern system design interviews.

Think of it as the **single entry point** for all client requests.

```text
Client
   |
   v
API Gateway
   |
   +---- User Service
   |
   +---- Order Service
   |
   +---- Payment Service
   |
   +---- Notification Service
```

Instead of clients talking directly to dozens of backend services, they communicate through the API Gateway.

---

# Real-Life Analogy

Imagine a hotel.

You don't directly contact:

* Housekeeping
* Kitchen
* Laundry
* Security

You call the reception desk.

Reception routes your request to the correct department.

The reception desk = API Gateway

---

# Why Do We Need API Gateway?

Without API Gateway:

```text
Mobile App
    |
    +---- User Service
    |
    +---- Order Service
    |
    +---- Payment Service
    |
    +---- Inventory Service
```

Problems:

### Too many endpoints

Client must know:

```text
users.company.com
orders.company.com
payments.company.com
inventory.company.com
```

Complex.

---

### Authentication everywhere

Every service must:

* Validate JWT
* Check permissions
* Verify tokens

Duplicate logic.

---

### Different APIs

Every service may expose APIs differently.

Client becomes tightly coupled.

---

### More network calls

To show one screen:

```text
Get User
Get Orders
Get Cart
Get Recommendations
```

Multiple calls.

Slower.

---

# With API Gateway

```text
Client
   |
   v
API Gateway
   |
   +---- User Service
   +---- Order Service
   +---- Payment Service
```

Client only knows:

```text
api.company.com
```

Gateway handles everything.

---

# Main Responsibilities of API Gateway

---

# 1. Request Routing

Most important job.

Gateway sends requests to correct service.

Example:

```http
/api/users/123
```

→ User Service

```http
/api/orders/123
```

→ Order Service

```http
/api/payments/123
```

→ Payment Service

Flow:

```text
Client
   |
   v
Gateway
   |
   +---- Route A → User Service
   |
   +---- Route B → Order Service
```

---

# 2. Authentication

Gateway validates:

* JWT tokens
* OAuth tokens
* API keys

Example:

```http
Authorization: Bearer xyz
```

Gateway verifies token.

Only valid requests move forward.

```text
Client
   |
Token Validation
   |
Allowed?
   |
  Yes
   |
Backend Service
```

Backend services become simpler.

---

# 3. Authorization

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What can you do?
```

Example:

```text
Admin -> allowed
Normal user -> denied
```

Gateway can enforce permissions.

---

# 4. Rate Limiting

Prevents abuse.

Example:

```text
100 requests/minute
```

If exceeded:

```http
429 Too Many Requests
```

Protects backend systems.

---

## Example

Without rate limiting:

```text
Attacker
   |
10,000 requests/sec
   |
Backend crashes
```

With gateway:

```text
Gateway
   |
Allows only 100/sec
```

System survives.

---

# 5. Load Balancing

Gateway can distribute traffic.

```text
Gateway
   |
   +---- Server 1
   +---- Server 2
   +---- Server 3
```

Round Robin:

```text
Req1 -> S1
Req2 -> S2
Req3 -> S3
Req4 -> S1
```

Improves scalability.

---

# 6. SSL/TLS Termination

HTTPS requests arrive encrypted.

```text
Client
   |
HTTPS
   |
Gateway
```

Gateway decrypts request.

Then forwards internally.

```text
HTTPS
   |
Gateway
   |
HTTP
   |
Services
```

Benefits:

* Easier certificate management
* Less CPU load on services

---

# 7. Request Aggregation

One client request can trigger multiple backend calls.

Example:

Home page needs:

```text
User Info
Orders
Recommendations
```

Without gateway:

```text
Client -> User Service
Client -> Order Service
Client -> Recommendation Service
```

3 network calls.

With gateway:

```text
Client
   |
One Request
   |
Gateway
   |
   + User Service
   + Order Service
   + Recommendation Service
```

Gateway combines results.

Returns one response.

---

# 8. Caching

Frequently requested data can be cached.

Example:

```text
Product Catalog
Country List
Configurations
```

Instead of:

```text
Gateway -> Service every time
```

Gateway returns cache.

Benefits:

* Lower latency
* Lower DB load

---

# 9. Request/Response Transformation

Gateway can modify requests.

Example:

Client sends:

```json
{
  "name":"Jay"
}
```

Gateway converts format.

Or adds headers.

```http
X-User-Id: 123
```

Useful during migrations.

---

# 10. Logging and Monitoring

Gateway sees every request.

Can collect:

```text
Request count
Error rate
Latency
Traffic patterns
```

Example metrics:

```text
QPS
P95 latency
P99 latency
5xx errors
```

Important for observability.

---

# API Gateway in Microservices

Very common architecture.

```text
                    +----------------+
                    | API Gateway    |
                    +--------+-------+
                             |
        -----------------------------------------
        |           |           |               |
        v           v           v               v

   User Svc   Order Svc   Payment Svc   Inventory Svc
```

Benefits:

* Hides service complexity
* Centralized security
* Easier client integration

---

# API Gateway vs Load Balancer

Many people confuse these.

### Load Balancer

Focus:

```text
Traffic distribution
```

Example:

```text
Request
   |
Load Balancer
   |
   + Server1
   + Server2
```

---

### API Gateway

Focus:

```text
Application-level routing
Authentication
Rate limiting
Caching
Aggregation
```

Example:

```text
Request
   |
API Gateway
   |
   + User Service
   + Payment Service
   + Order Service
```

---

# API Gateway vs Reverse Proxy

Reverse proxy forwards requests.

Examples:

* Nginx
* HAProxy

API Gateway is a more advanced reverse proxy.

Provides:

* Authentication
* Rate limiting
* Monitoring
* API management

You can think:

```text
API Gateway
    =
Reverse Proxy
+
Security
+
Routing
+
Monitoring
+
Rate Limiting
```

---

# Common API Gateway Products

### Cloud

* AWS API Gateway
* Azure API Management
* Google API Gateway

### Open Source

* Kong
* NGINX
* Traefik
* Ambassador

### Service Mesh Related

* Envoy Proxy

---

# Interview Example

### Design Amazon

User opens app.

Gateway receives:

```text
GET /homepage
```

Gateway:

```text
→ User Service
→ Product Service
→ Recommendation Service
→ Inventory Service
```

Aggregates response.

Returns:

```json
{
  "user":{},
  "products":[],
  "recommendations":[]
}
```

Client receives everything in one call.

---

# HLD Interview One-Line Answer

**API Gateway is a centralized entry point for client requests that handles routing, authentication, authorization, rate limiting, caching, monitoring, and request aggregation before forwarding requests to backend services.**

---

# Quick Revision (30 Seconds)

```text
API Gateway = Front Door of Microservices

Responsibilities:
✓ Routing
✓ Authentication
✓ Authorization
✓ Rate Limiting
✓ Load Balancing
✓ TLS Termination
✓ Caching
✓ Request Aggregation
✓ Monitoring
✓ Request Transformation

Benefits:
✓ Simpler clients
✓ Centralized security
✓ Reduced network calls
✓ Better scalability
✓ Easier management
```

### Interview Tip

When asked **"Why API Gateway?"**, answer:

> "In a microservices architecture, API Gateway acts as the single entry point for clients. It centralizes routing, authentication, rate limiting, caching, monitoring, and request aggregation, reducing complexity for clients and backend services while improving scalability and security."
