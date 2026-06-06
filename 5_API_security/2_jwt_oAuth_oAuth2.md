# JWT (JSON Web Token)

JWT is a **token format** used to securely transmit information between parties.

Think of JWT as a **digital identity card**.

After login, instead of asking the user to enter credentials on every request, the server gives a JWT.

The user sends this JWT with every API request.

---

# Why JWT Exists

Without JWT:

```text
Request
   |
   v
Server
   |
Check Session in Redis
   |
Get User Info
```

Every request requires a session lookup.

With JWT:

```text
Request + JWT
      |
      v
Server verifies token
      |
      v
User identified
```

No database lookup needed.

---

# JWT Structure

A JWT looks like:

```text
xxxxx.yyyyy.zzzzz
```

Three parts:

```text
Header.Payload.Signature
```

Example:

```text
eyJhbGciOiJIUzI1NiJ9
.
eyJ1c2VySWQiOjEwMSwicm9sZSI6ImFkbWluIn0
.
abcxyzsignature
```

---

# Part 1: Header

Contains metadata.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Meaning:

```text
alg = Signing algorithm
typ = JWT
```

---

# Part 2: Payload

Contains claims (data).

Example:

```json
{
  "userId": 101,
  "name": "Jay",
  "role": "admin",
  "exp": 1750000000
}
```

Claims are pieces of information.

Common claims:

```text
sub = user id
iat = issued at
exp = expiry time
iss = issuer
aud = audience
```

---

# Part 3: Signature

Most important part.

Generated using:

```text
Hash(
 Header +
 Payload +
 Secret Key
)
```

Example:

```text
HMACSHA256(
 header.payload,
 secret
)
```

---

# Why Signature Matters

Suppose attacker changes:

```json
{
 "role":"user"
}
```

to

```json
{
 "role":"admin"
}
```

Signature becomes invalid.

Server detects tampering.

Request rejected.

---

# JWT Login Flow

```text
User
 |
 | Email + Password
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

---

# Request Flow

Client stores JWT.

Every request:

```http
Authorization: Bearer JWT_TOKEN
```

Server:

```text
Verify Signature
Check Expiry
Extract Claims
```

User authenticated.

---

# JWT in Microservices

```text
               JWT
User -----------------> API Gateway
                           |
            ----------------------------
            |            |             |
            v            v             v
       User Service  Order Service  Payment Service
```

Every service can verify JWT independently.

No Redis needed.

This is why JWT is very popular.

---

# JWT Advantages

### Stateless

No session storage.

```text
No Redis
No Session DB
```

---

### Scalable

Easy horizontal scaling.

```text
Server A
Server B
Server C
```

Any server can verify token.

---

### Fast

No session lookup.

---

### Microservice Friendly

Each service can validate JWT.

---

# JWT Problems

### Logout is Hard

Token remains valid until expiry.

Example:

```text
Token Expiry = 1 Hour
```

Even after logout, token may work.

---

### Revocation is Hard

Need blacklist storage.

---

### Token Size

JWT is larger than session IDs.

---

# Access Token + Refresh Token

Most common architecture.

```text
Access Token
15 minutes

Refresh Token
30 days
```

Flow:

```text
Access Token Expired
        |
        v
Refresh Token
        |
        v
New Access Token
```

Used by:

```text
Google
Netflix
Spotify
Facebook
```

---

# OAuth

OAuth is **not authentication**.

OAuth is an **authorization framework**.

It allows one application to access another application's resources without sharing passwords.

---

# Real-Life Example

Suppose:

```text
Canva
```

wants access to:

```text
Google Drive
```

to import files.

Bad approach:

```text
Give Canva your Google password
```

Very dangerous.

OAuth solves this.

---

# OAuth Flow

```text
User
 |
 v
Canva
 |
 | Redirect
 |
 v
Google
 |
 | User Grants Permission
 |
 v
Google Returns Access Token
 |
 v
Canva Accesses Google Drive
```

User never shares password with Canva.

---

# OAuth Example

You click:

```text
Login with Google
```

or

```text
Continue with GitHub
```

Behind the scenes OAuth is used.

---

# OAuth Roles

Interview favorite.

---

## Resource Owner

Usually the user.

```text
You
```

---

## Client

Application requesting access.

```text
Canva
Spotify
Notion
```

---

## Authorization Server

Issues tokens.

```text
Google Auth Server
```

---

## Resource Server

Contains data.

```text
Google Drive
Google Calendar
```

---

# OAuth Analogy

Imagine:

```text
Hotel Room
```

Password approach:

```text
Give room key permanently
```

OAuth approach:

```text
Temporary access card
Limited permissions
```

Much safer.

---

# OAuth 1.0 vs OAuth 2.0

OAuth 2.0 is almost universally used today.

OAuth 1.0:

```text
Complex
Signature-heavy
```

OAuth 2.0:

```text
Simpler
Token-based
Industry standard
```

---

# OAuth 2.0 Authorization Code Flow

Most important flow for interviews.

---

## Step 1

User clicks:

```text
Login with Google
```

---

## Step 2

Redirect to Google.

```text
https://accounts.google.com
```

---

## Step 3

User logs into Google.

---

## Step 4

Google asks:

```text
Allow Canva to:

✓ Read Drive files
✓ Read Profile
```

User approves.

---

## Step 5

Google sends Authorization Code.

```text
auth_code_123
```

---

## Step 6

Canva exchanges code for token.

```text
Authorization Code
          |
          v
Access Token
```

---

## Step 7

Canva accesses Google APIs.

```http
Authorization: Bearer ACCESS_TOKEN
```

---

# OAuth 2.0 Flow Diagram

```text
User
 |
 | Login with Google
 |
 v
Client App
 |
 | Redirect
 |
 v
Google Auth Server
 |
 | Authorization Code
 |
 v
Client App
 |
 | Exchange Code
 |
 v
Access Token
 |
 v
Google APIs
```

---

# OAuth Scopes

Scopes define permissions.

Examples:

```text
read_profile
read_email
drive.read
calendar.read
```

Example:

```text
Canva only gets drive.read
```

Not full Google access.

---

# JWT vs OAuth

Very common interview confusion.

| JWT                     | OAuth                           |
| ----------------------- | ------------------------------- |
| Token format            | Authorization framework         |
| Stores claims           | Defines access delegation       |
| Used for authentication | Used for authorization          |
| Contains user info      | Defines how tokens are obtained |
| Example: JWT token      | Example: Login with Google      |

---

# Relationship Between JWT and OAuth

They are often used together.

OAuth says:

```text
How to obtain token
```

JWT says:

```text
What token looks like
```

Example:

```text
OAuth Flow
     |
     v
Access Token Issued
     |
     v
Token Format = JWT
```

---

# OpenID Connect (OIDC)

OAuth alone only handles authorization.

For authentication:

```text
OAuth + Identity Layer
```

called:

```text
OpenID Connect (OIDC)
```

This is what powers:

```text
Login with Google
Login with Microsoft
Login with GitHub
```

---

# HLD Interview Summary

### JWT

* A signed token format
* Contains user claims
* Stateless authentication
* Excellent for microservices
* Consists of Header + Payload + Signature

### OAuth 2.0

* Authorization framework
* Lets third-party apps access resources safely
* No password sharing
* Uses Access Tokens
* Most common flow: Authorization Code Flow

### One-line Difference

> **JWT is a token format, while OAuth 2.0 is a framework that defines how applications obtain and use tokens. OAuth can issue JWT tokens, but JWT itself is not OAuth.**
