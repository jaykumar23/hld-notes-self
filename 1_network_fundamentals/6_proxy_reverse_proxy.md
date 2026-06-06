# Proxy vs Reverse Proxy (Very Important HLD Interview Topic)

This is one of the most commonly asked networking topics in System Design interviews.

Many HLD components such as:

```text
CDN
API Gateway
Load Balancer
NGINX
Cloudflare
Kong
Envoy
```

are built using proxy concepts.

---

# First Understand: What is a Proxy?

A **proxy** is a server that sits in between two parties and forwards requests.

Instead of:

```text
Client --------> Server
```

we get:

```text
Client ---> Proxy ---> Server
```

The proxy receives the request first and then forwards it.

---

# Types of Proxy

There are two major types:

```text
1. Forward Proxy
2. Reverse Proxy
```

Most interview confusion happens here.

---

# Forward Proxy (Usually Called "Proxy")

### Location

```text
Client ---> Proxy ---> Internet
```

The proxy represents the client.

---

# Example

Suppose a company blocks direct internet access.

```text
Employee Laptop
        |
        v
   Corporate Proxy
        |
        v
     Google
```

Employee cannot directly access Google.

Every request goes through proxy.

---

# Flow

User opens:

```text
google.com
```

Request:

```text
Client
   |
   v
Proxy
   |
   v
Google
```

Google sees:

```text
Proxy IP
```

not the user's IP.

---

# Why Use Forward Proxy?

## 1. Hide Client Identity

Server sees:

```text
Proxy IP
```

instead of:

```text
Real User IP
```

---

## 2. Content Filtering

Companies block:

```text
Facebook
Instagram
YouTube
```

using proxy rules.

---

## 3. Security

Prevent direct internet access.

---

## 4. Caching

Frequently accessed pages stored.

```text
Employee1 -> Google Logo
Employee2 -> Cached Logo
```

Faster.

---

# Forward Proxy Diagram

```text
         Internet

            |
            v

      +-----------+
      |  Proxy    |
      +-----------+
         /   \
        /     \
 User1       User2
```

Proxy protects clients.

---

# Reverse Proxy

Much more important in HLD interviews.

---

## Location

```text
Client ---> Reverse Proxy ---> Servers
```

Reverse proxy represents servers.

---

# Example

Suppose Netflix has:

```text
Server1
Server2
Server3
```

Users should not access servers directly.

Architecture:

```text
Users
   |
Reverse Proxy
   |
------------
|    |     |
S1   S2    S3
```

Users talk only to proxy.

---

# Flow

User requests:

```http
GET /movies
```

Request reaches:

```text
NGINX
```

Reverse proxy decides:

```text
Server2
```

and forwards request.

---

# User Thinks

```text
Talking to Netflix
```

Actually:

```text
Talking to Reverse Proxy
```

---

# Reverse Proxy Diagram

```text
           Client
              |
              v

      +----------------+
      | Reverse Proxy  |
      +----------------+
         /    |     \
        /     |      \
      App1   App2   App3
```

Reverse proxy protects servers.

---

# Forward Proxy vs Reverse Proxy

| Feature      | Forward Proxy   | Reverse Proxy   |
| ------------ | --------------- | --------------- |
| Represents   | Client          | Server          |
| Sits Near    | Client          | Server          |
| Hides        | Client Identity | Server Identity |
| Used By      | Users           | Organizations   |
| Common Tools | Squid           | NGINX, Envoy    |

---

# Why Reverse Proxy is Used in HLD

This is where interviews focus.

---

# 1. Load Balancing

Without reverse proxy:

```text
User -> App Server
```

One server overloaded.

---

With reverse proxy:

```text
Users
  |
NGINX
  |
---------
|   |   |
S1  S2  S3
```

Traffic distributed.

Algorithms:

```text
Round Robin
Least Connections
Weighted
IP Hash
```

---

# Example

Requests:

```text
R1 -> Server1
R2 -> Server2
R3 -> Server3
R4 -> Server1
```

Load spread evenly.

---

# 2. SSL/TLS Termination

Common HLD interview question.

Instead of:

```text
Every server handles TLS
```

Use reverse proxy:

```text
Client
   |
 HTTPS
   |
Reverse Proxy
(decrypt)
   |
 HTTP
   |
Backend Servers
```

Benefits:

```text
Lower CPU
Simpler certificate management
```

---

# 3. Caching

Reverse proxy stores popular responses.

Example:

```text
GET /logo.png
```

First request:

```text
Proxy -> Backend
```

Second request:

```text
Proxy -> Cache
```

No backend call.

---

# 4. Security

Backend servers hidden.

Instead of:

```text
Public Internet
     |
App Servers
```

Use:

```text
Public Internet
      |
Reverse Proxy
      |
Private Servers
```

Attack surface reduced.

---

# 5. Compression

Reverse proxy compresses responses.

Example:

```text
2MB JSON
```

Compressed to:

```text
200KB
```

Less bandwidth.

---

# 6. Rate Limiting

Protects servers.

Example:

```text
User sends
10000 req/sec
```

Reverse proxy blocks:

```text
429 Too Many Requests
```

before backend is affected.

---

# 7. Authentication

Reverse proxy can verify:

```text
JWT
OAuth
API Keys
```

before request reaches backend.

---

# Reverse Proxy + Load Balancer

Many interviewers use both terms interchangeably.

Example:

```text
Internet
    |
Load Balancer
(Reverse Proxy)
    |
-------------
|     |     |
App1 App2 App3
```

AWS ALB works this way.

---

# NGINX Example

Most common reverse proxy.

Architecture:

```text
Users
  |
NGINX
  |
----------------
|      |       |
App1  App2   App3
```

Responsibilities:

```text
Load Balancing
SSL Termination
Caching
Compression
Routing
```

---

# API Gateway (Microservices)

API Gateway is basically an advanced reverse proxy.

Example:

```text
Client
   |
API Gateway
   |
--------------------
|      |      |
User  Order Payment
```

Gateway routes:

```text
/user/* -> User Service

/order/* -> Order Service

/payment/* -> Payment Service
```

---

# CDN as Reverse Proxy

Cloudflare:

```text
User
  |
Cloudflare
  |
Origin Server
```

Cloudflare acts as a reverse proxy.

Provides:

```text
Caching
Security
DDoS Protection
```

---

# Real Production Architecture

```text
User
  |
DNS
  |
CDN
  |
Load Balancer
  |
Reverse Proxy (NGINX)
  |
Application Servers
  |
Database
```

Interviewers love this diagram.

---

# Common HLD Questions

### Why use reverse proxy?

Answer:

```text
Load Balancing
Caching
SSL Termination
Security
Rate Limiting
Compression
Routing
```

---

### NGINX is what?

```text
Reverse Proxy
Web Server
Load Balancer
```

---

### API Gateway vs Reverse Proxy?

```text
API Gateway
=
Specialized Reverse Proxy
```

with:

```text
Auth
Rate Limiting
Monitoring
Service Discovery
```

---

### Load Balancer vs Reverse Proxy?

Many load balancers are reverse proxies.

Load balancing is one feature of a reverse proxy.

---

# Interview Architecture to Remember

```text
Client
   |
DNS
   |
CDN
   |
Load Balancer / Reverse Proxy
   |
App Servers
   |
Cache (Redis)
   |
Database
```

---

# 1-Minute Revision

```text
Forward Proxy:
Client -> Proxy -> Internet

Purpose:
Hide Client
Filtering
Security
Caching

Reverse Proxy:
Client -> Reverse Proxy -> Servers

Purpose:
Hide Servers
Load Balancing
Caching
SSL Termination
Security
Rate Limiting

Forward Proxy protects clients.
Reverse Proxy protects servers.

Examples:
Forward Proxy -> Corporate Proxy

Reverse Proxy ->
NGINX
Cloudflare
API Gateway
AWS ALB
Envoy
```

### Memory Trick

```text
Forward Proxy:
"I am the client."

Reverse Proxy:
"I am the server."
```

If the proxy stands in front of users → Forward Proxy.

If the proxy stands in front of servers → Reverse Proxy.
