# Session-Based Authentication vs Token-Based Authentication (JWT)

This is one of the most common HLD interview questions because almost every system needs authentication.

---

# First Understand the Goal

When a user logs in:

```text
Email + Password
```

Server verifies credentials.

Now the server must remember:

```text
"This user is already logged in"
```

Otherwise the user would need to enter the password on every request.

There are two major approaches:

```text
1. Session-Based Authentication
2. Token-Based Authentication (JWT)
```

---

# Session-Based Authentication

Traditional approach.

### Login Flow

```text
User
 |
 | Username + Password
 |
 v
Server
 |
 | Verify credentials
 |
 v
Create Session
 |
 v
Store Session in DB/Redis
 |
 v
Return Session ID
```

Example:

```text
Session ID = ABC123
```

Server stores:

```text
ABC123 → User 101
```

---

## Request Flow

User sends:

```http
Cookie: SESSION_ID=ABC123
```

Server receives:

```text
ABC123
```

Looks up:

```text
ABC123 → User 101
```

User authenticated.

---

## Architecture

```text
Client
  |
  | Session ID
  |
  v
Server
  |
  | Lookup Session
  |
  v
Redis / DB
```

---

# Real-Life Analogy

Imagine a hotel.

After check-in:

```text
Reception gives Room Card #ABC123
```

Hotel keeps record:

```text
ABC123 → Room 501
```

Whenever you enter:

```text
Show card
```

Hotel checks database.

That's session authentication.

---

# Session Storage Problem

Suppose:

```text
Server A
Server B
Server C
```

User logs into:

```text
Server A
```

Session stored only on:

```text
Server A
```

Next request goes to:

```text
Server B
```

Problem:

```text
Session not found
```

User appears logged out.

---

# Solution 1: Sticky Sessions

Load balancer always sends user to same server.

```text
User
  |
  v
LB
  |
  +----> Server A
```

Problem:

```text
Poor load distribution
```

Not ideal.

---

# Solution 2: Central Session Store

Store sessions in:

```text
Redis
```

Architecture:

```text
            Redis
              ^
              |
Server A -----|
Server B -----|
Server C -----|
```

Any server can verify session.

---

# Session Authentication Pros

### Easy Logout

Delete session.

```text
Redis:
ABC123 → Deleted
```

User instantly logged out.

---

### Easy Revocation

Admin can kill session.

```text
Delete session
```

Done.

---

### More Secure by Default

Sensitive data remains on server.

Client only stores:

```text
Session ID
```

---

# Session Authentication Cons

### Extra Database Lookup

Every request:

```text
Session ID
 ↓
Redis Lookup
 ↓
Get User
```

Extra network call.

---

### Server State Required

Need:

```text
Redis
Database
Session Store
```

---

### Scaling Complexity

Millions of users:

```text
Millions of sessions
```

Need distributed session storage.

---

# Token-Based Authentication (JWT)

Modern approach.

Instead of storing session on server:

```text
Store user information inside token.
```

---

# Login Flow

```text
User
 |
 | Login
 |
 v
Auth Server
 |
 | Verify Credentials
 |
 v
Generate JWT
 |
 v
Return JWT
```

Example:

```json
{
  "userId": 101,
  "role": "admin",
  "exp": 1710000000
}
```

---

# JWT Structure

Looks like:

```text
xxxxx.yyyyy.zzzzz
```

Three parts:

```text
Header
Payload
Signature
```

---

## Header

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

---

## Payload

```json
{
  "userId": 101,
  "role": "admin"
}
```

---

## Signature

Generated using secret key.

```text
Signature =
Hash(Header + Payload + Secret)
```

Prevents tampering.

---

# Request Flow

Client sends:

```http
Authorization: Bearer JWT_TOKEN
```

Server:

```text
Verify Signature
Check Expiry
```

If valid:

```text
Authenticated
```

No database lookup needed.

---

# Architecture

```text
Client
  |
  | JWT
  |
  v
API Gateway
  |
  v
Microservices
```

Each service verifies token independently.

---

# Real-Life Analogy

Imagine airport boarding pass.

Boarding pass already contains:

```text
Name
Flight
Seat Number
Gate
```

Airport doesn't ask database every time.

Information is already printed.

JWT works similarly.

---

# Why JWT Is Popular in Microservices

Suppose:

```text
User Service
Order Service
Payment Service
Inventory Service
```

With sessions:

```text
Every service needs Redis lookup
```

With JWT:

```text
Verify token locally
```

Much faster.

---

# Session vs JWT Request Flow

## Session

```text
Request
  |
Session ID
  |
  v
Redis Lookup
  |
  v
User Info
```

---

## JWT

```text
Request
  |
JWT
  |
  v
Verify Signature
  |
  v
User Info Available
```

No Redis call.

---

# Performance Comparison

### Session

```text
Network Call Required
```

```text
App
 ↓
Redis
```

Latency added.

---

### JWT

```text
Local Verification
```

No extra network trip.

Lower latency.

---

# Logout Difference

## Session

Very Easy

```text
Delete Session
```

User logged out instantly.

---

## JWT

Harder

Token exists until expiry.

Example:

```text
Expiry = 1 hour
```

User may remain authenticated.

---

# JWT Logout Solutions

### Short Expiry

```text
Access Token = 15 min
```

Common.

---

### Blacklist

Store revoked tokens.

```text
Redis Blacklist
```

Problem:

```text
Need lookup again
```

---

### Refresh Token Pattern

Most common.

---

# Access Token + Refresh Token

```text
Access Token
 15 min

Refresh Token
 30 days
```

Flow:

```text
Access Token Expires
      |
      v
Use Refresh Token
      |
      v
Get New Access Token
```

Used by:

```text
Google
Facebook
Netflix
Spotify
```

---

# Security Comparison

## Session

Client stores:

```text
Session ID
```

Actual user data stays server-side.

Safer.

---

## JWT

Contains claims:

```json
{
  "userId": 101,
  "role": "admin"
}
```

Never store:

```text
Password
Credit Card
Sensitive Data
```

inside JWT.

---

# HLD Interview Tradeoff Table

| Feature            | Session       | JWT       |
| ------------------ | ------------- | --------- |
| Server State       | Yes           | No        |
| DB/Redis Lookup    | Every request | No        |
| Scalability        | Moderate      | Excellent |
| Logout             | Easy          | Hard      |
| Revocation         | Easy          | Hard      |
| Microservices      | Less suitable | Excellent |
| Performance        | Slower        | Faster    |
| Storage            | Server        | Client    |
| Horizontal Scaling | Harder        | Easier    |

---

# When To Use Sessions

Good for:

```text
Banking Applications
Admin Panels
Internal Enterprise Apps
Traditional Web Apps
```

Why?

```text
Easy logout
Easy revocation
More control
```

---

# When To Use JWT

Good for:

```text
Microservices
Mobile Apps
SPAs (React/Angular)
Large Distributed Systems
Public APIs
```

Why?

```text
Stateless
Highly scalable
No session storage
```

---

# What Big Companies Use?

Most modern systems use:

```text
JWT Access Token
+
Refresh Token
```

Architecture:

```text
User
  |
  v
Auth Service
  |
  +--> Access Token (15 min)
  |
  +--> Refresh Token (30 days)
```

This gives:

```text
Fast authentication
+
Scalability
+
Controlled logout
```

# HLD Interview One-Liner

**Session authentication stores login state on the server and uses a session ID on the client, while JWT/token authentication stores user claims inside a signed token and keeps the server stateless. Sessions provide easier logout and revocation, whereas JWTs scale better and are preferred in modern microservice architectures.**
