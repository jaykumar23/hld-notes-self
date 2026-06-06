# HTTP & HTTPS (Must Know for HLD Interviews)

Whenever you open:

```text
https://google.com
https://amazon.com
https://youtube.com
```

your browser communicates using **HTTP/HTTPS**.

In HLD interviews, topics like:

* APIs
* Load Balancers
* Microservices
* Authentication
* CDN
* Caching

all heavily use HTTP/HTTPS.

---

# What is HTTP?

**HTTP = HyperText Transfer Protocol**

It is an **Application Layer (OSI Layer 7)** protocol used for communication between:

```text
Client (Browser/App)
        ↔
Server
```

Example:

```text
Browser
   |
HTTP Request
   |
Server
   |
HTTP Response
   |
Browser
```

---

# Real Example

You open:

```text
https://amazon.com
```

Browser sends:

```http
GET / HTTP/1.1
Host: amazon.com
```

Server replies:

```http
HTTP/1.1 200 OK

<html>...</html>
```

---

# HTTP Request Structure

Example:

```http
GET /products/123 HTTP/1.1
Host: amazon.com
Authorization: Bearer xyz

Body (optional)
```

Contains:

### Request Line

```http
GET /products/123 HTTP/1.1
```

---

### Headers

```http
Host: amazon.com
Content-Type: application/json
Authorization: token
```

---

### Body

For POST/PUT requests.

Example:

```json
{
  "name": "Laptop"
}
```

---

# HTTP Response Structure

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
   "id":123
}
```

Contains:

### Status Line

```http
HTTP/1.1 200 OK
```

### Headers

```http
Content-Type: application/json
Cache-Control: max-age=3600
```

### Body

Actual data.

---

# HTTP Methods

## GET

Fetch data.

```http
GET /users/1
```

Example:

```text
View product
View profile
```

---

## POST

Create resource.

```http
POST /users
```

Example:

```text
Register User
Create Order
```

---

## PUT

Update entire resource.

```http
PUT /users/1
```

---

## PATCH

Partial update.

```http
PATCH /users/1
```

---

## DELETE

Delete resource.

```http
DELETE /users/1
```

---

# HTTP Status Codes

Interview favorite.

---

## 2xx Success

### 200 OK

```text
Request successful
```

### 201 Created

```text
Resource created
```

---

## 3xx Redirect

### 301

```text
Permanent redirect
```

### 302

```text
Temporary redirect
```

---

## 4xx Client Errors

### 400

```text
Bad Request
```

### 401

```text
Unauthorized
```

### 403

```text
Forbidden
```

### 404

```text
Not Found
```

---

## 5xx Server Errors

### 500

```text
Internal Server Error
```

### 502

```text
Bad Gateway
```

Often seen with load balancers.

### 503

```text
Service Unavailable
```

---

# Problem with HTTP

HTTP data is sent as plain text.

Example:

```http
Username: jay
Password: 12345
```

Anyone intercepting packets can read it.

This is a huge security risk.

---

# HTTPS

**HTTPS = HTTP + TLS/SSL Encryption**

HTTPS solves:

```text
Confidentiality
Integrity
Authentication
```

---

# HTTP vs HTTPS

| Feature       | HTTP | HTTPS |
| ------------- | ---- | ----- |
| Encryption    | No   | Yes   |
| Secure        | No   | Yes   |
| Port          | 80   | 443   |
| TLS/SSL       | No   | Yes   |
| Password Safe | No   | Yes   |

---

# How HTTPS Works

Suppose:

```text
Client
   |
Google Server
```

Before HTTP data is exchanged:

### TLS Handshake

A secure connection is established.

---

# Simplified TLS Flow

### Step 1

Client:

```text
Hello
Supported Cipher Suites
```

---

### Step 2

Server:

```text
Certificate
Public Key
```

---

### Step 3

Browser verifies certificate.

Issued by:

```text
CA (Certificate Authority)
```

Examples:

```text
DigiCert
Let's Encrypt
GlobalSign
```

---

### Step 4

Session key created.

---

### Step 5

All future communication encrypted.

```text
GET /orders
```

becomes unreadable encrypted bytes.

---

# Why Not Encrypt Every Packet Using Public Key?

Public-key encryption is expensive.

Instead:

```text
Public Key
      ↓
Exchange Secret Key
      ↓
Symmetric Encryption
```

for actual communication.

Much faster.

---

# SSL vs TLS

Interview question.

Technically:

```text
SSL = Old
TLS = Modern
```

People still say:

```text
SSL Certificate
```

but modern systems use TLS.

---

# Certificates

A certificate proves:

```text
You are really talking to Google
```

and not an attacker.

Contains:

```text
Domain Name
Public Key
Issuer
Expiry Date
```

---

# HLD: HTTPS Through Load Balancer

Common architecture:

```text
User
   |
HTTPS
   |
Load Balancer
   |
HTTP/HTTPS
   |
App Servers
```

---

## SSL Termination

Most companies decrypt at LB.

```text
Client
   |
HTTPS
   |
Load Balancer
(decrypt)
   |
HTTP
   |
App Servers
```

Benefits:

```text
Less CPU usage on servers
Centralized certificate management
```

---

# Keep-Alive

Without keep-alive:

```text
Request 1 -> New TCP Connection
Request 2 -> New TCP Connection
Request 3 -> New TCP Connection
```

Very expensive.

With keep-alive:

```text
One TCP Connection
      ↓
Many HTTP Requests
```

Huge performance improvement.

---

# HTTP/1.1 vs HTTP/2 vs HTTP/3

### HTTP/1.1

```text
One request at a time per connection
```

Older.

---

### HTTP/2

Introduced:

```text
Multiplexing
Header Compression
```

Multiple requests simultaneously.

Much faster.

---

### HTTP/3

Built on:

```text
QUIC
UDP
```

instead of TCP.

Benefits:

```text
Lower latency
Faster connection setup
Better mobile performance
```

Modern browsers use HTTP/3 extensively.

---

# HTTP in Microservices

Internal communication:

```text
API Gateway
      |
User Service
      |
Order Service
      |
Payment Service
```

Often uses:

```text
HTTP/HTTPS
gRPC
```

under the hood.

---

# Common HLD Questions

### Why HTTPS instead of HTTP?

```text
Encryption
Authentication
Integrity
```

---

### Why Port 443?

Standard HTTPS port.

---

### Why TLS handshake?

To establish a secure encrypted channel.

---

### Why SSL termination at LB?

```text
Reduce server CPU load
Centralized certificate management
```

---

### HTTP or TCP?

Interview trap.

```text
HTTP = Application Layer
TCP  = Transport Layer
```

HTTP usually runs on TCP.

```text
HTTP
  ↓
TCP
  ↓
IP
```

---

# Complete Flow of Opening Google

```text
1. User enters google.com

2. DNS resolves domain
   ↓
   IP Address

3. TCP 3-way handshake

4. TLS Handshake

5. HTTPS Request

GET /

6. Server Response

200 OK

7. Browser renders page
```

---

# 1-Minute HLD Revision

```text
HTTP = HyperText Transfer Protocol
HTTPS = HTTP + TLS Encryption

Ports:
HTTP  -> 80
HTTPS -> 443

Methods:
GET
POST
PUT
PATCH
DELETE

Common Status Codes:
200 OK
201 Created
301 Redirect
401 Unauthorized
403 Forbidden
404 Not Found
500 Server Error

HTTPS Benefits:
Encryption
Authentication
Integrity

Flow:
DNS
 ↓
TCP Handshake
 ↓
TLS Handshake
 ↓
HTTP Request/Response

Modern Versions:
HTTP/1.1
HTTP/2
HTTP/3 (QUIC over UDP)
```

### Interview Memory Line

```text
HTTP defines HOW applications communicate.
TCP ensures packets arrive.
TLS secures the communication.
HTTPS = HTTP running over TLS over TCP.
```
