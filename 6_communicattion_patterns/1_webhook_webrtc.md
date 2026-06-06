# Webhook Explained in Detail (HLD Interview)

A **Webhook** is a mechanism where one system automatically sends data to another system when an event happens.

Instead of continuously asking for updates (**polling**), the receiving system gets notified automatically.

---

# Simple Definition

Webhook = **Event-driven HTTP callback**

```text
Event Happens
      |
      v
Source System
      |
HTTP POST
      |
      v
Destination System
```

---

# Real-Life Analogy

Imagine you ordered food.

### Polling Approach

You keep calling the restaurant:

```text
Is my food ready?
Is my food ready?
Is my food ready?
```

Many unnecessary calls.

---

### Webhook Approach

You tell the restaurant:

```text
Call me when food is ready.
```

Restaurant calls only when needed.

That's exactly how webhooks work.

---

# Why Do We Need Webhooks?

Without webhooks:

```text
Your App --> Payment Gateway

Your App keeps checking:

Payment completed?
Payment completed?
Payment completed?
```

This is polling.

Problems:

```text
Extra API calls
More server load
Higher costs
More latency
```

---

With webhooks:

```text
Payment completed
       |
       v
Payment Gateway
       |
HTTP POST
       |
       v
Your Server
```

No continuous checking required.

---

# Basic Flow

Suppose Stripe payment succeeds.

```text
Customer
    |
    v
Stripe
```

Payment successful.

Stripe automatically sends:

```http
POST /webhook
```

to your server.

Payload:

```json
{
  "event":"payment_success",
  "orderId":"123",
  "amount":1000
}
```

Your server receives it and updates the order.

---

# Webhook Architecture

```text
Source System
(Stripe)

     |
     | POST
     v

Webhook Endpoint

     |
     v

Business Logic

     |
     v

Database
```

---

# Example 1: Payment Gateway

Most common interview example.

### Step 1

User pays.

```text
User
  |
  v
Stripe
```

---

### Step 2

Stripe processes payment.

```text
Success
```

---

### Step 3

Stripe sends webhook.

```http
POST /webhook/payment
```

---

### Step 4

Your system updates order.

```text
Order Status:
Pending -> Paid
```

---

# Example 2: GitHub Webhook

GitHub can notify CI/CD systems.

When code is pushed:

```text
Developer
   |
Push Code
   |
GitHub
   |
Webhook
   |
Jenkins
```

Jenkins starts build automatically.

---

# Example 3: E-commerce

Order shipped.

```text
Amazon
   |
Webhook
   |
SMS Service
```

SMS sent automatically.

---

# How Webhooks Work Internally

### Registration

First you provide a URL.

Example:

```text
https://myapp.com/webhook
```

---

Provider stores it.

```text
Stripe
   |
Stores URL
```

---

### Event Occurs

```text
Payment Success
```

---

### HTTP Request Sent

```http
POST /webhook

{
  "event":"payment_success"
}
```

---

### Receiver Processes Event

```text
Update Database
Send Email
Generate Invoice
```

---

# Polling vs Webhook

## Polling

```text
Client ---> Server
Client ---> Server
Client ---> Server
```

Repeated requests.

---

## Webhook

```text
Event Happens
      |
      v
Server ---> Client
```

Only when required.

---

| Feature     | Polling   | Webhook |
| ----------- | --------- | ------- |
| Requests    | Many      | Few     |
| Latency     | Higher    | Lower   |
| Cost        | Higher    | Lower   |
| Server Load | Higher    | Lower   |
| Real-time   | Not ideal | Better  |

---

# Webhook Request Example

```http
POST /webhook/payment

Content-Type: application/json
```

Body:

```json
{
  "event":"payment_success",
  "payment_id":"p123",
  "amount":500
}
```

---

# Security in Webhooks (Very Important)

Interviewers often ask:

> How do you secure webhooks?

---

## Problem

Anyone can call:

```http
POST /webhook
```

and send fake data.

---

## Solution 1: Signature Verification

Provider sends signature.

Header:

```http
X-Signature: abc123
```

Receiver verifies.

```text
Valid Signature?
      |
     Yes
      |
 Process
```

Otherwise reject.

---

Used by:

```text
Stripe
GitHub
PayPal
```

---

## Solution 2: Secret Key

Shared secret:

```text
my_secret_key
```

Provider signs payload.

Receiver validates.

---

## Solution 3: IP Whitelisting

Accept requests only from:

```text
Stripe IPs
GitHub IPs
```

---

## Solution 4: HTTPS

Always use:

```text
https://
```

Never:

```text
http://
```

Protects against snooping.

---

# Webhook Reliability Issues

Webhooks are not guaranteed to succeed.

Suppose:

```text
Stripe sends webhook
```

but:

```text
Your server is down
```

Request fails.

---

# Retry Mechanism

Most providers retry.

```text
Attempt 1
Attempt 2
Attempt 3
Attempt 4
```

until successful.

---

Example:

```text
Stripe
GitHub
PayPal
```

all retry failed webhooks.

---

# Idempotency (Extremely Important)

Because retries happen:

```text
Payment Success
```

may arrive multiple times.

Example:

```text
Webhook #1
Webhook #2
Webhook #3
```

Same event.

---

Bad:

```text
Order marked paid 3 times
```

Good:

```text
Process only once
```

---

Use:

```text
event_id
payment_id
transaction_id
```

to deduplicate.

---

# Webhook Delivery Pattern

Usually:

```text
At Least Once Delivery
```

Meaning:

```text
Message may arrive multiple times
```

but should never be lost.

---

Receiver must handle duplicates.

---

# Webhooks in Microservices

Service A:

```text
Order Created
```

can notify:

```text
Inventory Service
Notification Service
Analytics Service
```

using webhooks.

```text
Order Service
      |
      +--> Webhook
      |
      +--> Inventory
      |
      +--> Analytics
```

---

# Webhook vs API

Many people confuse these.

---

## API

You ask for data.

```text
Client --> Server
```

Example:

```http
GET /orders
```

---

## Webhook

Server pushes data.

```text
Server --> Client
```

Example:

```http
POST /webhook
```

---

| API                 | Webhook          |
| ------------------- | ---------------- |
| Pull model          | Push model       |
| Client initiates    | Server initiates |
| Request when needed | Event-driven     |
| Polling possible    | No polling       |

---

# HLD Interview Example

### Design Payment Processing System

Flow:

```text
User
 |
Payment Gateway
 |
Payment Success
 |
Webhook
 |
Order Service
 |
Update DB
 |
Publish Kafka Event
 |
Notification Service
```

This is a production-grade design.

---

# Interview Cheat Sheet

### Webhook

> A webhook is an event-driven HTTP callback where one system automatically sends data to another system when a specific event occurs.

### Key Points

* Push-based communication
* Uses HTTP POST
* Event-driven
* Reduces polling
* Supports near real-time updates
* Requires retries
* Must be idempotent
* Secure using signatures and HTTPS

### Common Examples

* Stripe payment success
* GitHub push events
* PayPal transaction updates
* Shopify order updates
* Slack notifications
* CI/CD build triggers

A common HLD interview answer is:

> "Instead of polling the payment gateway for transaction status, I'll use a webhook so the gateway can notify our system asynchronously when the payment succeeds. The webhook endpoint will verify the signature, process the event idempotently, and publish an event to Kafka for downstream services."


| Feature                 | Webhook                 | WebSocket              |
| ----------------------- | ----------------------- | ---------------------- |
| Communication           | One-way                 | Two-way                |
| Connection              | Temporary               | Persistent             |
| Protocol                | HTTP POST               | WebSocket Protocol     |
| Real-time               | Near Real-Time          | Real-Time              |
| Client can send anytime | No                      | Yes                    |
| Server can send anytime | Only during callback    | Yes                    |
| Scalability             | Easier                  | More complex           |
| Connection overhead     | Low                     | High                   |
| Common Use Cases        | Payments, GitHub events | Chat, Gaming, Tracking |

# WebRTC Explained in Detail for HLD Interviews

WebRTC is one of the most important topics for designing:

* Zoom
* Google Meet
* Microsoft Teams
* WhatsApp Calls
* FaceTime
* Discord Voice Channels
* Live Collaboration Apps

---

# What is WebRTC?

**WebRTC = Web Real-Time Communication**

It is a technology that enables:

```text
Audio
Video
Screen Sharing
Data Transfer
```

between browsers and applications in real time.

---

## Simple Definition

> WebRTC allows two devices to communicate directly with low latency without requiring media to pass through the application server.

---

# Why Not Use Normal HTTP?

HTTP works like:

```text
Request
Response
Connection Closed
```

Not suitable for:

```text
Video Calls
Voice Calls
Gaming
Screen Sharing
```

because data must flow continuously.

---

# Why Not Use WebSockets?

WebSockets can send messages.

```text
Client <----> Server
```

Good for:

```text
Chat
Notifications
```

But video generates huge traffic.

Example:

```text
720p Video
=
Several MB/sec
```

Routing everything through your application server becomes expensive.

---

# Goal of WebRTC

Instead of:

```text
User A
   |
Server
   |
User B
```

WebRTC tries:

```text
User A <---------> User B
```

Direct communication.

This is called:

```text
Peer-to-Peer (P2P)
```

---

# High-Level Architecture

```text
         Signaling Server
               |
               |
User A ---------------- User B
        WebRTC
```

The server helps users find each other.

Actual media may flow directly.

---

# Important Components

Interviewers often ask:

> Explain WebRTC architecture.

There are 4 major components.

---

## 1. Signaling Server

Most important concept.

---

Before users can talk:

```text
User A must know:
- User B exists
- Network information
- Supported codecs
```

Need a coordination server.

```text
User A
    |
Signaling Server
    |
User B
```

---

### Signaling Server Responsibilities

Exchange:

```text
IP Address
Port
Media Capabilities
Session Information
```

---

Usually implemented using:

```text
WebSocket
Socket.IO
gRPC
```

---

Example:

```text
Zoom Server
```

helps establish connection.

---

### Important Interview Point

Signaling is NOT part of WebRTC.

You build it yourself.

---

# 2. STUN Server

One of the most frequently asked interview topics.

---

Problem:

Users sit behind NAT.

Example:

```text
Home Router
Corporate Firewall
```

User thinks:

```text
IP = 192.168.1.10
```

But public internet sees:

```text
IP = 49.207.x.x
```

Need to discover public address.

---

STUN means:

```text
Session Traversal Utilities for NAT
```

---

Flow:

```text
User
   |
STUN Server
   |
Returns Public IP
```

---

Example:

```text
Private IP:
192.168.1.5

Public IP:
49.207.20.11
```

Now peer can attempt connection.

---

# 3. TURN Server

Another favorite interview topic.

---

Sometimes direct connection fails.

Reasons:

```text
Strict Firewall
Corporate Network
Symmetric NAT
```

---

Then:

```text
User A  X  User B
```

Cannot connect directly.

---

Solution:

```text
TURN Server
```

TURN means:

```text
Traversal Using Relays around NAT
```

---

Architecture:

```text
User A
    |
TURN
    |
User B
```

Media goes through TURN.

---

### Downside

Expensive.

Because:

```text
All video traffic
flows through server
```

Huge bandwidth cost.

---

Companies try:

```text
STUN first
TURN only if needed
```

---

# 4. ICE

ICE combines everything.

ICE means:

```text
Interactive Connectivity Establishment
```

---

Its job:

```text
Find best route
between peers
```

---

It tries:

```text
1. Direct connection
2. STUN route
3. TURN route
```

Automatically.

---

# Complete Connection Flow

Interview-ready flow:

---

## Step 1

User A joins call.

```text
User A
```

---

## Step 2

User B joins call.

```text
User B
```

---

## Step 3

Signaling exchange.

```text
A -> Signaling Server -> B
```

Exchange:

```text
SDP
Network Info
Codecs
```

---

## Step 4

STUN discovers public IPs.

```text
A -> STUN
B -> STUN
```

---

## Step 5

ICE attempts connection.

```text
Direct Path?
```

---

## Step 6

If direct works:

```text
A <------> B
```

---

## Step 7

If direct fails:

```text
A -> TURN -> B
```

---

# SDP (Session Description Protocol)

Another interview keyword.

SDP contains:

```text
Audio Codec
Video Codec
Network Details
Encryption Info
```

Example:

```text
Supports:
VP8
H264
Opus Audio
```

---

# Media Flow

After connection:

```text
Audio
Video
Screen Share
```

flow directly.

---

Uses:

```text
UDP
```

mostly.

---

Why UDP?

Video calls prefer:

```text
Speed > Perfect Reliability
```

Missing one frame is okay.

Waiting for retransmission is bad.

---

# Why UDP Instead of TCP?

TCP:

```text
Reliable
Slower
```

UDP:

```text
Fast
Low Latency
```

---

Video call example:

Missing one frame:

```text
User won't notice
```

Delay of 2 seconds:

```text
Terrible experience
```

Hence UDP.

---

# WebRTC Security

WebRTC is encrypted by default.

Uses:

```text
DTLS
SRTP
```

---

Media encryption:

```text
Audio
Video
Screen Share
```

---

# Scaling Challenges

Suppose:

```text
2 Users
```

Easy.

```text
A <--> B
```

---

What about:

```text
100 participants?
```

---

# Pure Peer-to-Peer Problem

Each user sends video to everyone.

For N users:

```text
N × (N-1)
```

connections.

---

Example:

```text
100 users
```

Need:

```text
9900 connections
```

Impossible.

---

# SFU (Most Important HLD Topic)

Modern systems use SFU.

SFU =

```text
Selective Forwarding Unit
```

---

Architecture:

```text
        SFU
      /  |  \
     /   |   \
    A    B    C
```

Users send one stream.

SFU forwards streams.

---

Benefits:

```text
Low CPU
Low Bandwidth
Scalable
```

---

Used by:

```text
Zoom
Google Meet
Teams
```

---

# MCU

Older approach.

MCU =

```text
Multipoint Control Unit
```

---

Architecture:

```text
Users
   |
 MCU
   |
Mixed Video
```

Server mixes all videos.

---

Pros:

```text
Simple client
```

Cons:

```text
Huge server cost
```

---

Rare today.

---

# SFU vs MCU

| Feature      | SFU         | MCU    |
| ------------ | ----------- | ------ |
| Mixing       | No          | Yes    |
| Server CPU   | Low         | High   |
| Latency      | Low         | Higher |
| Scalability  | Better      | Worse  |
| Modern Usage | Very Common | Rare   |

---

# HLD: Design Zoom

Architecture:

```text
Users
   |
Load Balancer
   |
Signaling Servers
   |
SFU Cluster
   |
Recording Service
   |
Storage
```

---

Flow:

```text
1. User joins
2. Signaling exchanges SDP
3. STUN/TURN setup
4. Connect to SFU
5. Audio/video forwarded
6. Recording stored in cloud
```

---

# Common Interview Questions

### Why STUN?

```text
Discover public IP
```

---

### Why TURN?

```text
Fallback relay
when direct connection fails
```

---

### Why UDP?

```text
Lower latency
```

---

### Why Signaling Server?

```text
Exchange connection info
```

---

### Why SFU?

```text
Supports large meetings efficiently
```

---

# Quick Revision Sheet

```text
WebRTC = Real-time Audio/Video Communication

Core Components:
---------------
Signaling Server
STUN
TURN
ICE

Connection Flow:
---------------
Signaling
→ STUN
→ ICE
→ Direct P2P
→ TURN fallback

Protocols:
----------
UDP
DTLS
SRTP

Scaling:
--------
2 users → P2P

Many users → SFU

Zoom/Meet Architecture:
----------------------
Client
→ Signaling Server
→ SFU Cluster
→ Storage/Recording
```

### 30-Second Interview Answer

> WebRTC is a real-time communication technology used for audio, video, and screen sharing. It establishes peer-to-peer connections using a signaling server for connection setup, STUN for public IP discovery, TURN as a relay when direct connections fail, and ICE to choose the best network path. For large-scale systems like Zoom or Google Meet, media is typically routed through SFUs rather than pure peer-to-peer connections to achieve scalability and low latency.

