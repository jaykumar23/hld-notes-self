# TCP vs UDP (Very Important HLD Interview Topic)

Both TCP and UDP are **Transport Layer (Layer 4)** protocols.

When one machine sends data to another:

```text
Application (HTTP, DNS, etc.)
          ↓
TCP / UDP
          ↓
IP
          ↓
Network
```

The biggest difference:

```text
TCP = Reliable
UDP = Fast
```

---

# Quick Comparison

| Feature         | TCP                           | UDP                          |
| --------------- | ----------------------------- | ---------------------------- |
| Full Form       | Transmission Control Protocol | User Datagram Protocol       |
| Connection      | Connection-Oriented           | Connectionless               |
| Reliability     | Reliable                      | Unreliable                   |
| Packet Ordering | Guaranteed                    | Not Guaranteed               |
| Acknowledgment  | Yes                           | No                           |
| Retransmission  | Yes                           | No                           |
| Speed           | Slower                        | Faster                       |
| Header Size     | 20+ bytes                     | 8 bytes                      |
| Use Cases       | HTTP, HTTPS, Banking, DB      | Gaming, Streaming, VoIP, DNS |

---

# TCP (Transmission Control Protocol)

TCP guarantees:

```text
Data arrives
Data arrives once
Data arrives in order
```

Example:

```text
Send:
A → B → C → D

Receive:
A → B → C → D
```

No missing packets.

No out-of-order packets.

---

# How TCP Works

Before sending data:

## 3-Way Handshake

Client and Server establish a connection.

### Step 1

Client:

```text
SYN
```

Meaning:

```text
Can we connect?
```

---

### Step 2

Server:

```text
SYN + ACK
```

Meaning:

```text
Yes, let's connect.
```

---

### Step 3

Client:

```text
ACK
```

Meaning:

```text
Connection established.
```

---

```text
Client                 Server

 SYN  ------------>

      <--------- SYN+ACK

 ACK  ------------>

Connection Ready
```

This is why TCP has a small setup cost.

---

# Reliability in TCP

Suppose:

```text
Packet1
Packet2
Packet3
```

Packet2 gets lost.

TCP notices:

```text
Packet1 ✓
Packet2 ✗
Packet3 ✓
```

Then:

```text
Retransmit Packet2
```

This makes TCP reliable.

---

# Packet Ordering

Suppose network delivers:

```text
Packet3
Packet1
Packet2
```

TCP rearranges:

```text
Packet1
Packet2
Packet3
```

before giving data to application.

---

# Flow Control

Imagine:

```text
Server → Very Fast
Client → Slow
```

Without control:

```text
Client buffer overflow
```

TCP uses a **receive window** to control speed.

```text
Send only what I can handle.
```

---

# Congestion Control

Suppose millions of packets flood the network.

TCP automatically slows down.

Algorithms:

```text
Slow Start
AIMD
CUBIC
BBR
```

Google uses BBR heavily.

---

# TCP Use Cases

Use TCP when data loss is unacceptable.

Examples:

### Banking

```text
Transfer ₹10,000
```

Cannot lose packet.

---

### HTTPS

```text
Open Amazon
Open Google
Open Gmail
```

Uses TCP.

---

### Databases

```text
MySQL
PostgreSQL
MongoDB replication
```

Need reliable delivery.

---

# UDP (User Datagram Protocol)

UDP is much simpler.

No:

```text
Connection
Acknowledgment
Ordering
Retransmission
Flow Control
```

Just:

```text
Send packet
Hope it arrives
```

---

# UDP Flow

```text
Client
   |
Send Packet
   |
Server
```

No handshake.

No connection setup.

Very fast.

---

# Example

Send:

```text
A
B
C
D
```

Receiver may get:

```text
A
C
D
```

or

```text
C
A
D
```

or

```text
A
B
```

UDP doesn't care.

---

# Why Use UDP Then?

Because speed matters more than perfection.

---

## Live Video Streaming

Suppose:

```text
Frame 100 lost
```

Would you rather:

### TCP

Wait for retransmission

```text
Video freezes
```

Bad experience.

---

### UDP

Skip frame

```text
Video continues smoothly
```

Better experience.

---

# Gaming Example

Online game:

```text
Player Position
x=100
y=200
```

Next update:

```text
x=102
y=201
```

If one packet is lost:

No problem.

Latest position matters.

UDP is preferred.

---

# VoIP / Video Calls

Examples:

```text
Zoom
Google Meet
WhatsApp Call
Discord
```

Use UDP heavily.

A lost packet is better than delayed audio.

---

# DNS Uses UDP

DNS lookup:

```text
google.com ?
```

Response:

```text
142.250.xxx.xxx
```

Small message.

Fast response needed.

UDP is ideal.

(Modern DNS can fall back to TCP when necessary.)

---

# TCP vs UDP in Real Applications

| Application     | Protocol     |
| --------------- | ------------ |
| HTTP            | TCP          |
| HTTPS           | TCP          |
| SMTP            | TCP          |
| SSH             | TCP          |
| FTP             | TCP          |
| MySQL           | TCP          |
| PostgreSQL      | TCP          |
| DNS             | UDP (mostly) |
| Video Streaming | UDP          |
| Online Gaming   | UDP          |
| VoIP            | UDP          |
| Live Broadcast  | UDP          |

---

# HLD Interview Example

### Design WhatsApp Messaging

Message:

```text
Hello
```

Must arrive exactly once.

Use:

```text
TCP
```

---

### Design WhatsApp Call

Voice packets:

```text
Audio Stream
```

Need low latency.

Use:

```text
UDP
```

---

# TCP vs UDP in Terms of Latency

TCP:

```text
Handshake
ACK
Retransmission
Ordering
```

More latency.

---

UDP:

```text
No handshake
No ACK
No ordering
```

Lower latency.

---

# Easy Memory Trick

Think of:

## TCP = Registered Courier

```text
Package Sent
Signature Taken
Tracking Available
Resend if Lost
```

Reliable but slower.

---

## UDP = Throwing Flyers

```text
Send quickly
No tracking
No guarantee
```

Fast but unreliable.

---

# Most Asked Interview Question

### When would you choose UDP over TCP?

Answer:

```text
When low latency is more important than reliability.
```

Examples:

```text
Gaming
Video Calls
Live Streaming
VoIP
DNS
```

### When would you choose TCP?

Answer:

```text
When correctness and reliability are critical.
```

Examples:

```text
Payments
Banking
Databases
Web Applications
File Transfers
```

---

# 30-Second Revision

```text
TCP:
✓ Reliable
✓ Ordered
✓ ACKs
✓ Retransmission
✓ Connection-Oriented

Examples:
HTTPS
Banking
Databases

UDP:
✓ Fast
✓ Low Latency
✗ No ACK
✗ No Retransmission
✗ No Ordering

Examples:
Gaming
Streaming
VoIP
DNS

Rule:
Need Reliability → TCP
Need Speed → UDP
```
