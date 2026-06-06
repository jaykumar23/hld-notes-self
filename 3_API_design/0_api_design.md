# 1. What is an API?

API = **Application Programming Interface**

An API is simply a **contract that allows two software systems to communicate**.

Think of it like a restaurant:

```text
Customer  -> API -> Kitchen
```

* Customer = Client (Browser/Mobile App)
* API = Waiter
* Kitchen = Backend Server

You don't directly talk to the kitchen.

You place a request through the waiter (API).

---

## Example

Request:

```http
GET /users/123
```

Response:

```json
{
  "id": 123,
  "name": "Jay"
}
```

The client asks for user information.

The server returns data.

---

## Why APIs Matter in HLD

Almost every system communicates through APIs.

Examples:

```text
Frontend → Backend
Backend → Database
Backend → Payment Service
Microservice → Microservice
```

Interview Point:

> APIs define how components interact in a distributed system.

---

# 2. Data Formats (JSON, Protobuf, Avro)

When systems communicate, they need a common format.

---

# JSON

Most common format.

Example:

```json
{
  "id": 1,
  "name": "John"
}
```

Advantages:

✅ Human readable

✅ Easy debugging

✅ Works everywhere

Disadvantages:

❌ Large size

❌ Slower parsing

---

## Example Size

```json
{
  "id":123,
  "name":"Jay"
}
```

Might take:

```text
25 bytes+
```

---

# Protobuf (Protocol Buffers)

Created by Google.

Data is converted into compact binary form.

Schema:

```proto
message User {
  int32 id = 1;
  string name = 2;
}
```

Advantages:

✅ Very small payload

✅ Fast serialization

✅ Fast deserialization

Disadvantages:

❌ Not human readable

❌ Requires schema definition

---

## Used In

```text
gRPC
Microservices
Internal communication
```

---

# Avro

Created by Apache.

Schema-based serialization.

Example schema:

```json
{
 "type":"record",
 "name":"User",
 "fields":[
   {"name":"id","type":"int"},
   {"name":"name","type":"string"}
 ]
}
```

Advantages:

✅ Schema evolution

✅ Efficient storage

✅ Big Data friendly

---

## Used In

```text
Kafka
Hadoop
Data Pipelines
```

---

## HLD Comparison

| Format   | Readable | Size  | Speed  | Use         |
| -------- | -------- | ----- | ------ | ----------- |
| JSON     | Yes      | Large | Medium | Public APIs |
| Protobuf | No       | Small | Fast   | gRPC        |
| Avro     | No       | Small | Fast   | Kafka/Data  |

Interview Answer:

> JSON is preferred for external APIs because it's human-readable. Protobuf is preferred for service-to-service communication because it is smaller and faster. Avro is common in data pipelines and Kafka ecosystems.

---

# 3. API Architectural Styles

Different ways of designing APIs.

Main ones:

```text
REST
GraphQL
gRPC
SOAP
```

Modern interviews focus on:

```text
REST
GraphQL
gRPC
```

---

# REST

Resource-based architecture.

Everything is a resource.

Example:

```text
/users
/orders
/products
```

Operations:

```http
GET
POST
PUT
DELETE
```

---

# GraphQL

Client requests exactly what it needs.

Example:

```graphql
{
 user {
   name
   email
 }
}
```

---

# gRPC

Remote Procedure Calls over HTTP/2.

Example:

```protobuf
GetUser(123)
```

Feels like calling a local function.

---

# HLD Rule

```text
Public API → REST
Flexible UI → GraphQL
Internal Microservices → gRPC
```

---

# 4. REST API Design

REST = Representational State Transfer

Most common API style.

---

## Resources

Bad:

```text
/getUser
/createUser
```

Good:

```text
/users
/users/123
```

---

## HTTP Methods

### GET

Read data

```http
GET /users/123
```

---

### POST

Create data

```http
POST /users
```

---

### PUT

Replace resource

```http
PUT /users/123
```

---

### PATCH

Partial update

```http
PATCH /users/123
```

---

### DELETE

Delete resource

```http
DELETE /users/123
```

---

## Status Codes

### Success

```text
200 OK
201 Created
204 No Content
```

---

### Client Errors

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
```

---

### Server Errors

```text
500 Internal Server Error
503 Service Unavailable
```

---

## REST Best Practices

### Use nouns

```text
/users
/orders
/products
```

not

```text
/getUsers
/createUsers
```

---

### Version APIs

```text
/v1/users
/v2/users
```

---

### Stateless

Each request contains everything needed.

Server doesn't remember previous requests.

---

# 5. GraphQL Deep Dive

Created by Facebook.

Client specifies exactly what data it wants.

---

## REST Problem

Frontend wants:

```text
Name
Email
```

REST response:

```json
{
 "name":"Jay",
 "email":"abc@gmail.com",
 "phone":"123",
 "address":"Mumbai",
 "dob":"..."
}
```

Extra fields are wasted.

Called:

```text
Over-fetching
```

---

## Another Problem

Need:

```text
User
Posts
Comments
```

May require:

```text
GET /user
GET /posts
GET /comments
```

Called:

```text
Under-fetching
```

---

# GraphQL Solution

Single query:

```graphql
{
 user(id:1){
   name
   email
   posts{
      title
   }
 }
}
```

Only required data is returned.

---

## Advantages

✅ Single endpoint

```text
/graphql
```

✅ Prevents over-fetching

✅ Flexible

---

## Disadvantages

❌ Complex caching

❌ Harder backend implementation

❌ Heavy queries can become expensive

---

## HLD Use Cases

```text
Facebook
GitHub
Mobile Apps
Complex dashboards
```

---

# 6. gRPC Deep Dive

gRPC = Google Remote Procedure Call

Designed for high-performance service communication.

---

## REST

```http
GET /users/123
```

---

## gRPC

```protobuf
GetUser(id=123)
```

Looks like a function call.

---

# How It Works

Step 1

Define contract:

```proto
service UserService {
  rpc GetUser(UserRequest)
      returns(UserResponse);
}
```

---

Step 2

Generate client/server code.

---

Step 3

Call RPC method.

---

# Why Fast?

Uses:

```text
HTTP/2
Protobuf
Binary Serialization
```

instead of:

```text
HTTP/1.1 + JSON
```

---

# HTTP/2 Features

## Multiplexing

Multiple requests on one connection.

```text
Req1
Req2
Req3
```

Same TCP connection.

---

## Header Compression

Smaller network payload.

---

## Streaming

Client Stream

```text
Client -> Server
```

Server Stream

```text
Server -> Client
```

Bidirectional Stream

```text
Client <-> Server
```

---

# Advantages

✅ Extremely fast

✅ Low latency

✅ Strong contracts

✅ Streaming support

---

# Disadvantages

❌ Browser support limited

❌ Debugging harder

❌ Not human-readable

---

## HLD Use Cases

```text
Google
Netflix
Uber
Microservices
```

---

# REST vs GraphQL vs gRPC

| Feature          | REST      | GraphQL | gRPC      |
| ---------------- | --------- | ------- | --------- |
| Human Readable   | Yes       | Yes     | No        |
| Flexible Queries | No        | Yes     | No        |
| Speed            | Medium    | Medium  | Fast      |
| Binary           | No        | No      | Yes       |
| Streaming        | Limited   | No      | Yes       |
| Public APIs      | Excellent | Good    | Poor      |
| Microservices    | Good      | Medium  | Excellent |

---

# 7. API Versioning Strategies

APIs change over time.

Need versioning so old clients don't break.

---

## URL Versioning

Most common.

```text
/v1/users
/v2/users
```

Advantages:

✅ Easy

✅ Explicit

---

## Header Versioning

```http
Accept-Version: v2
```

---

## Query Parameter Versioning

```http
/users?version=v2
```

---

## HLD Recommendation

Use:

```text
URL Versioning
```

Most widely used and easy to manage.

---

# 8. Idempotency

Very important interview topic.

---

## Definition

An operation is idempotent if:

> Performing it multiple times produces the same final result.

---

## Example

Delete User:

```http
DELETE /users/1
```

First call:

```text
User deleted
```

Second call:

```text
Still deleted
```

Final state same.

Therefore:

```text
Idempotent
```

---

## HTTP Methods

### Idempotent

```text
GET
PUT
DELETE
```

---

### Non-Idempotent

```text
POST
```

Example:

```http
POST /orders
```

Calling twice:

```text
Order #101
Order #102
```

Creates duplicates.

---

# Idempotency Key

Used in payments.

Request:

```http
POST /payment
Idempotency-Key: abc123
```

Retry:

```http
POST /payment
Idempotency-Key: abc123
```

Server recognizes duplicate request.

Only one payment is created.

---

## HLD Example

```text
Stripe
PayPal
Razorpay
```

All use idempotency keys.

---

# 9. Pagination and Filtering

Large datasets cannot be returned at once.

---

# Pagination

Instead of:

```text
1 million users
```

Return:

```text
100 users
```

per request.

---

## Offset Pagination

```http
/users?page=2&limit=100
```

Database:

```sql
LIMIT 100 OFFSET 100
```

---

Advantages:

✅ Easy

Disadvantages:

❌ Slow for large offsets

---

# Cursor Pagination

Request:

```http
/users?cursor=abc123
```

Response:

```json
{
  "users": [...],
  "nextCursor": "xyz456"
}
```

---

Advantages:

✅ Very scalable

✅ No duplicate records

✅ Fast

---

Used by:

```text
Twitter
Facebook
Instagram
LinkedIn
```

---

# Filtering

Allow clients to narrow results.

Example:

```http
/users?city=Mumbai
```

---

Multiple Filters

```http
/products?category=laptop&brand=dell
```

---

Sorting

```http
/products?sort=price
```

Descending:

```http
/products?sort=-price
```

---

# Interview Summary (Must Remember)

```text
API = Communication contract between systems

JSON = Human-readable
Protobuf = Fast + Small
Avro = Kafka/Data Pipelines

REST = Resource-based public APIs
GraphQL = Client chooses fields
gRPC = Fast microservice communication

Version APIs using /v1, /v2

Idempotency = Same operation repeated safely

POST payments should use Idempotency Keys

Offset Pagination = Simple
Cursor Pagination = Scalable

Filtering reduces unnecessary data transfer
```

These are the API concepts most frequently asked before moving into larger HLD topics like API Gateway, Microservices, Service Discovery, Rate Limiting, Caching, and Load Balancers.
