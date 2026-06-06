# OSI Model (Very Important for HLD & System Design)

The **OSI (Open Systems Interconnection) Model** is a conceptual framework that explains **how data travels from one computer to another over a network**.

In HLD interviews, you won't usually be asked to recite all 7 layers, but understanding them helps in topics like:

* Load Balancers
* DNS
* TCP/IP
* HTTP/HTTPS
* Reverse Proxies
* Firewalls
* CDNs
* WebSockets
* API Communication

---

# Quick Memory Trick

```text
7. Application
6. Presentation
5. Session
4. Transport
3. Network
2. Data Link
1. Physical
```

Data moves:

```text
Application
   ↓
Presentation
   ↓
Session
   ↓
Transport
   ↓
Network
   ↓
Data Link
   ↓
Physical
```

Across network...

Then reverse at receiver.

---

# Real Example

Suppose:

```text
Your Browser
     ↓
Google Server
```

You type:

```text
https://www.google.com
```

and press Enter.

The request travels through all OSI layers.

---

# Layer 7: Application Layer

### Purpose

Provides services directly to applications.

Examples:

```text
HTTP
HTTPS
FTP
SMTP
DNS
SSH
WebSocket
```

When browser sends:

```http
GET /search?q=hello
```

Application layer creates the request.

### HLD Relevance

Interviewers constantly discuss:

```text
REST APIs
GraphQL
DNS
WebSockets
gRPC
```

All are Application Layer concepts.

---

# Layer 6: Presentation Layer

### Purpose

Data formatting, encryption, compression.

Responsible for:

```text
Encryption
Decryption
Encoding
Decoding
Compression
```

Example:

Before sending:

```text
Hello
```

Convert into:

```text
UTF-8 bytes
```

HTTPS:

```text
TLS/SSL encryption
```

happens here conceptually.

### HLD Relevance

When interviewer asks:

```text
How HTTPS works?
```

Encryption comes from this layer.

---

# Layer 5: Session Layer

### Purpose

Maintains communication sessions.

Example:

```text
User logs in
Session starts
User sends requests
Session maintained
Logout
Session ends
```

Manages:

```text
Authentication Session
Connection Session
Token-based Session
```

### HLD Relevance

Interview questions:

```text
How user sessions work?
How sticky sessions work?
```

Related to Session Layer concepts.

---

# Layer 4: Transport Layer

One of the most important layers for interviews.

---

## Purpose

Reliable end-to-end communication.

Protocols:

```text
TCP
UDP
```

---

## TCP

Reliable

Features:

```text
Acknowledgment
Retransmission
Ordering
Error Detection
Flow Control
```

Example:

```text
Banking
Payments
Database Replication
HTTPS
```

Uses TCP.

---

## UDP

Fast but unreliable.

No:

```text
Acknowledgment
Ordering
Retransmission
```

Example:

```text
Video Streaming
Gaming
VoIP
DNS
```

---

## Port Numbers

Transport Layer identifies application.

Examples:

```text
80   HTTP
443  HTTPS
22   SSH
3306 MySQL
5432 PostgreSQL
```

---

### HLD Relevance

Interviewers ask:

```text
TCP vs UDP?
```

Very common.

---

# Layer 3: Network Layer

Responsible for routing packets across networks.

Protocol:

```text
IP (Internet Protocol)
```

Works with:

```text
IPv4
IPv6
Routers
```

---

## IP Address

Example:

```text
142.250.183.14
```

Every machine gets an address.

Router decides:

```text
Which path packet should take?
```

---

### HLD Relevance

Topics:

```text
Routing
Internet
CDN
Multi-region architecture
```

depend heavily on Layer 3.

---

# Layer 2: Data Link Layer

Responsible for communication inside local network.

Uses:

```text
MAC Address
Switches
Ethernet
```

Example:

```text
Laptop → WiFi Router
```

Router identifies device using MAC address.

Example MAC:

```text
00:1A:2B:3C:4D:5E
```

---

### HLD Relevance

Rarely asked directly.

Useful for:

```text
ARP
LAN
Switches
```

---

# Layer 1: Physical Layer

Actual transmission of bits.

Medium:

```text
Fiber Cable
Ethernet Cable
Radio Signals
WiFi Signals
```

Sends:

```text
0s and 1s
```

as electrical/optical/radio signals.

---

# Complete Flow of Opening Google

### Step 1

Browser creates:

```http
GET / HTTP/1.1
Host: google.com
```

Application Layer.

---

### Step 2

HTTPS encrypts request.

Presentation Layer.

---

### Step 3

Session established.

Session Layer.

---

### Step 4

TCP creates segments.

```text
Source Port: 50000
Destination Port: 443
```

Transport Layer.

---

### Step 5

IP added.

```text
Source IP: Your Machine
Destination IP: Google
```

Network Layer.

---

### Step 6

MAC addresses added.

Data Link Layer.

---

### Step 7

Bits sent via:

```text
WiFi/Fiber
```

Physical Layer.

---

# Encapsulation (Interview Favorite)

Each layer adds its own header.

```text
Application Data
      ↓
TCP Header + Data
      ↓
IP Header + TCP Data
      ↓
MAC Header + IP Data
      ↓
Bits on wire
```

Example:

```text
[MAC][IP][TCP][HTTP DATA]
```

Receiver removes them in reverse order.

---

# OSI vs TCP/IP Model

In real systems, TCP/IP is used more.

OSI:

```text
7 Layers
```

TCP/IP:

```text
Application
Transport
Internet
Network Access
```

Mapping:

```text
OSI Layer 7,6,5 → Application

OSI Layer 4 → Transport

OSI Layer 3 → Internet

OSI Layer 2,1 → Network Access
```

---

# Most Important Layers for HLD Interviews

Focus heavily on:

### Layer 7 (Application)

```text
HTTP
HTTPS
DNS
gRPC
REST
WebSockets
```

### Layer 4 (Transport)

```text
TCP
UDP
Ports
Load Balancers
```

### Layer 3 (Network)

```text
IP
Routing
CDN
Internet
```

These three layers appear in almost every System Design interview.

---

# 30-Second HLD Revision

```text
L7 Application  → HTTP, DNS, APIs
L6 Presentation → Encryption, Compression
L5 Session      → Session Management
L4 Transport    → TCP, UDP, Ports
L3 Network      → IP, Routing
L2 Data Link    → MAC, Switches
L1 Physical     → Wires, Fiber, WiFi

Encapsulation:
HTTP → TCP → IP → MAC → Bits

Most important for HLD:
Application + Transport + Network
```
