# Checksums (Important Networking & HLD Concept)

A **checksum** is a small value calculated from data and sent along with it to detect whether the data was corrupted during transmission.

Think of it as a **data integrity check**.

---

# Why Do We Need Checksums?

Suppose a sender transmits:

```text
HELLO
```

Across the network:

```text
Sender --------> Receiver
```

Due to noise, hardware issues, or transmission errors, the receiver might get:

```text
HELMO
```

How does the receiver know the data changed?

Answer:

```text
Checksum
```

---

# Real-Life Analogy

Imagine sending a package.

You write:

```text
Items Count = 5
```

Receiver opens the package and counts:

```text
Items Count = 4
```

Immediately they know something is wrong.

Checksum works similarly for digital data.

---

# Basic Idea

Sender:

```text
Data
   ↓
Calculate Checksum
   ↓
Send Data + Checksum
```

Receiver:

```text
Receive Data + Checksum
   ↓
Recalculate Checksum
   ↓
Compare
```

If:

```text
Checksums Match
```

Data is likely correct.

If:

```text
Checksums Don't Match
```

Data was corrupted.

---

# Simple Example

Suppose data is:

```text
1 2 3 4
```

Sender computes:

```text
1 + 2 + 3 + 4 = 10
```

Checksum:

```text
10
```

Send:

```text
Data: 1 2 3 4
Checksum: 10
```

---

Receiver gets:

```text
1 2 3 4
Checksum: 10
```

Calculates:

```text
1+2+3+4 = 10
```

Match:

```text
Valid
```

---

Now suppose transmission error:

```text
1 2 8 4
Checksum: 10
```

Receiver computes:

```text
1+2+8+4 = 15
```

Mismatch:

```text
Corrupted Data
```

Detected.

---

# Internet Checksum (TCP/UDP)

TCP and UDP packets contain checksum fields.

Simplified packet:

```text
+------------------+
| Header           |
+------------------+
| Data             |
+------------------+
| Checksum         |
+------------------+
```

Sender calculates checksum over:

```text
Header + Data
```

Receiver verifies it.

---

# Where Checksums Are Used?

## TCP

Every TCP segment includes a checksum.

```text
TCP Header
TCP Payload
Checksum
```

If corrupted:

```text
Packet discarded
```

TCP can request retransmission.

---

## UDP

UDP also has a checksum.

If corrupted:

```text
Packet discarded
```

But UDP does **not retransmit**.

Data is simply lost.

---

## IP Header

IPv4 has header checksum.

Used to detect corruption in:

```text
IP Header
```

(Not the payload.)

---

# Example: TCP Reliability

Suppose sender sends:

```text
Packet #5
```

Packet becomes corrupted.

Receiver:

```text
Checksum Failed
```

Drops packet.

TCP notices missing acknowledgment:

```text
ACK not received
```

Retransmits:

```text
Packet #5
```

This is part of why TCP is reliable.

---

# Checksum vs Hash

Interviewers sometimes ask this.

### Checksum

Goal:

```text
Error Detection
```

Examples:

```text
TCP Checksum
UDP Checksum
```

Fast.

Not cryptographically secure.

---

### Hash

Goal:

```text
Integrity Verification
Security
```

Examples:

```text
SHA-256
MD5
SHA-512
```

Used in:

```text
Passwords
Digital Signatures
File Verification
```

---

Example:

```text
File Download
```

Website may provide:

```text
SHA-256 Hash
```

to verify file wasn't modified.

---

# Checksum vs CRC

Another interview question.

### Checksum

Simple arithmetic.

Example:

```text
Add bytes together
```

Fast.

Less accurate.

---

### CRC (Cyclic Redundancy Check)

Uses polynomial math.

Much better at detecting errors.

Used in:

```text
Ethernet
Storage Devices
Network Frames
```

---

# TCP Checksum Flow

```text
Sender
  |
Data
  |
Checksum Generated
  |
Packet Sent
  |
Network
  |
Receiver
  |
Checksum Recomputed
  |
Compare
```

If:

```text
Match
```

Accept packet.

If:

```text
Mismatch
```

Drop packet.

---

# Why Checksums Matter in HLD

Most HLD interviews won't ask you to calculate one manually.

But you should know:

### TCP Reliability

```text
Checksum
+
ACK
+
Retransmission
```

work together.

---

### Data Integrity

Checksums help detect:

```text
Corrupted packets
Transmission errors
Hardware issues
```

---

### Storage Systems

Distributed systems often use:

```text
Checksums
Hashes
```

to verify:

```text
Replication
Backups
Data Consistency
```

Examples:

```text
HDFS
Cassandra
Kafka
S3
```

---

# Interview Questions

### Q1: What is a checksum?

A value computed from data to detect corruption during transmission.

---

### Q2: Does checksum guarantee data is correct?

No.

It only makes corruption detection highly likely.

Rare collisions can occur.

---

### Q3: What happens if TCP checksum fails?

```text
Packet dropped
```

Sender eventually retransmits.

---

### Q4: Does UDP use checksum?

Yes.

But corrupted packets are discarded without retransmission.

---

### Q5: Checksum vs Hash?

```text
Checksum → Error detection

Hash → Security and integrity verification
```

---

# 30-Second Revision

```text
Checksum = Small value computed from data

Purpose:
Detect transmission errors

Flow:
Sender:
Data → Checksum → Send

Receiver:
Recompute Checksum
Compare

Match:
✓ Accept

Mismatch:
✗ Corrupted

Used In:
TCP
UDP
IPv4 Header

TCP:
Checksum + ACK + Retransmission

Checksum vs Hash:
Checksum → Error detection
Hash → Integrity/Security
```

### Memory Line

```text
A checksum is a quick fingerprint of data used to detect accidental corruption during transmission.
```
