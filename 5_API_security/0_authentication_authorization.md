# Authentication vs Authorization (Very Important HLD Interview Topic)

These two terms are asked in almost every System Design / HLD interview.

Many candidates confuse them, but they solve different problems.

---

# Simple Definition

## Authentication (AuthN)

**Authentication = Verifying who you are**

The system checks your identity.

Example:

```text
Username + Password
OTP
Google Login
Fingerprint
Face ID
JWT Verification
```

Question answered:

```text
"Who are you?"
```

---

## Authorization (AuthZ)

**Authorization = Verifying what you can do**

After identity is verified, the system checks permissions.

Question answered:

```text
"What are you allowed to do?"
```

Example:

```text
Admin can delete users

Normal user cannot delete users

Manager can approve expenses

Employee cannot approve expenses
```

---

# Real World Example

Imagine entering an airport.

### Authentication

Security checks your passport.

```text
You are Jaykumar
Identity verified
```

### Authorization

Now they check:

```text
Can you enter VIP lounge?

Can you access cockpit?

Can you enter security area?
```

Those depend on permissions.

---

# Authentication Flow

```text
User
 |
 | Login Request
 |
 v
Auth Service
 |
 | Verify Credentials
 |
 v
Database
 |
 v
Identity Confirmed
 |
 v
Token Generated
```

Example:

```text
Email = abc@gmail.com
Password = password123
```

System verifies credentials.

If correct:

```text
JWT Token Issued
```

---

# Authorization Flow

```text
User
 |
 | Access Admin API
 |
 v
Server
 |
 | Check Permissions
 |
 v
Allow / Deny
```

Example:

```http
DELETE /users/123
```

System checks:

```text
Role = Admin ?
```

If yes:

```text
200 OK
```

Else:

```text
403 Forbidden
```

---

# Authentication Happens Before Authorization

Always:

```text
Authentication
       ↓
Authorization
```

You cannot decide permissions until you know who the user is.

Example:

```text
Unknown person
```

System cannot determine permissions.

Identity must be verified first.

---

# Common Authentication Methods

## 1. Username + Password

```text
Most common
```

Flow:

```text
User enters password
Server hashes password
Compare with stored hash
```

Never store plain passwords.

Store:

```text
bcrypt(password)
```

or

```text
Argon2(password)
```

---

## 2. OTP Authentication

```text
SMS OTP
Email OTP
```

Flow:

```text
Send OTP
Verify OTP
Authenticate User
```

Used in:

```text
Banking
UPI Apps
```

---

## 3. Social Login

```text
Google Login
GitHub Login
Facebook Login
```

Flow:

```text
Redirect to Google
Google verifies user
Google sends token
```

Based on:

```text
OAuth 2.0
OpenID Connect
```

---

## 4. Biometric Authentication

```text
Fingerprint
Face ID
Retina Scan
```

Used mainly on devices.

---

# Session-Based Authentication

Traditional websites.

Flow:

```text
Login
Server creates session
Session stored in Redis/DB
Session ID returned
```

```text
Client
   |
Session ID Cookie
   |
Server
```

---

### Example

```text
Session ID = ABC123
```

Stored:

```text
ABC123 → User 101
```

Advantages:

```text
Easy logout
Easy revocation
```

Disadvantages:

```text
Need server-side storage
Scaling challenge
```

---

# Token-Based Authentication (JWT)

Most modern systems use JWT.

Flow:

```text
Login
 ↓
Generate JWT
 ↓
Return JWT
 ↓
Client stores token
 ↓
Send token on every request
```

Example:

```http
Authorization: Bearer JWT_TOKEN
```

JWT contains:

```json
{
  "userId": 101,
  "role": "admin",
  "exp": 1710000000
}
```

Server verifies signature.

No DB lookup required.

---

# JWT Authentication Architecture

```text
        Login
User -----------> Auth Server
                     |
                     |
                  JWT
                     |
                     v
                  Client
                     |
                     | JWT
                     v
               API Gateway
                     |
                     v
              Microservices
```

Common HLD architecture.

---

# Authorization Models

---

# 1. RBAC (Role Based Access Control)

Most common.

Permissions assigned to roles.

```text
Admin
Manager
Employee
Guest
```

Example:

```text
Admin
 ├── Create User
 ├── Delete User
 └── View Reports

Employee
 └── View Reports
```

---

### Database Design

```text
Users
Roles
Permissions
```

Tables:

```text
users
roles
permissions
role_permissions
user_roles
```

---

### Example

```text
Jay → Admin
```

Admin permissions:

```text
Create User
Delete User
View Logs
```

---

# 2. ABAC (Attribute Based Access Control)

Uses attributes.

Example:

```text
Department
Location
Designation
Time
```

Rule:

```text
Employee.department = Finance
AND
Location = Mumbai
```

Allow access.

Used in enterprises.

---

# 3. ACL (Access Control List)

Object stores permission list.

Example:

```text
Document A

User1 -> Read
User2 -> Write
User3 -> Delete
```

Used in:

```text
Google Drive
File Systems
```

---

# Authentication vs Authorization Table

| Feature        | Authentication   | Authorization        |
| -------------- | ---------------- | -------------------- |
| Purpose        | Verify identity  | Verify permissions   |
| Question       | Who are you?     | What can you do?     |
| Happens First? | Yes              | After authentication |
| Example        | Login            | Access control       |
| Output         | User identity    | Allowed actions      |
| Failure Code   | 401 Unauthorized | 403 Forbidden        |

---

# HTTP Status Codes (Interview Favorite)

## Authentication Failure

```http
401 Unauthorized
```

Meaning:

```text
Identity not verified
```

Examples:

```text
Missing JWT
Invalid JWT
Expired JWT
Wrong Password
```

---

## Authorization Failure

```http
403 Forbidden
```

Meaning:

```text
Identity known
Permission denied
```

Example:

```text
User is authenticated

But not an admin
```

---

# Authentication & Authorization in Microservices

```text
          User
            |
            v
      API Gateway
            |
     Verify JWT
            |
            v
    User Service
    Order Service
    Payment Service
```

Authentication often happens at:

```text
API Gateway
```

Authorization can happen at:

```text
Gateway
or
Service Level
```

Example:

```text
Only admin can access /admin/*
```

Gateway blocks request early.

---

# HLD Interview Answer (30 Seconds)

> Authentication verifies the identity of a user ("Who are you?"), using methods like passwords, OTPs, sessions, or JWTs. Authorization determines what actions that authenticated user is allowed to perform ("What can you do?"), typically using RBAC, ABAC, or ACL models. Authentication happens first, and failures usually return 401, while authorization failures return 403. In modern distributed systems, authentication is often handled by an Auth Service and JWTs, while authorization is enforced by API Gateways and microservices.
