# IP Addressing (Important HLD Interview Topic)

When interviewers discuss:

* Client → Server communication
* Load Balancers
* Microservices
* DNS
* CDN
* Multi-region deployments
* Kubernetes networking

IP addressing is the foundation.

---

# What is an IP Address?

An **IP (Internet Protocol) Address** is a unique identifier assigned to a device on a network.

Think of it as:

```text
Home Address → House
IP Address → Computer/Server
```

Without an IP address, data packets don't know where to go.

Example:

```text
142.250.183.14
```

(Google server IP)

---

# Why Do We Need IP Addresses?

Suppose your browser sends:

```http
GET https://google.com
```

How does the internet know where Google's server is?

Answer:

```text
DNS converts google.com
        ↓
IP Address
        ↓
142.250.xxx.xxx
```

Packets are then routed to that IP.

---

# IPv4

Most common IP format.

Example:

```text
192.168.1.10
```

Structure:

```text
192 . 168 . 1 . 10
 ↑     ↑    ↑   ↑
8bit 8bit 8bit 8bit
```

Total:

```text
32 bits
```

Each block:

```text
0 - 255
```

Range:

```text
0.0.0.0
to
255.255.255.255
```

---

# How Many IPv4 Addresses Exist?

IPv4:

```text
32 bits
```

Therefore:

```text
2^32
=
4,294,967,296
```

≈ 4.3 billion addresses

Not enough for modern internet.

---

# IPv6

Created because IPv4 addresses ran out.

Example:

```text
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

Uses:

```text
128 bits
```

Total:

```text
2^128
```

Huge number.

Enough for virtually unlimited devices.

---

# Public vs Private IP

Very important interview topic.

---

## Public IP

Visible on internet.

Example:

```text
8.8.8.8
```

(Google DNS)

Can be reached globally.

---

## Private IP

Used inside local networks.

Examples:

```text
10.0.0.0 - 10.255.255.255

172.16.0.0 - 172.31.255.255

192.168.0.0 - 192.168.255.255
```

Example:

```text
Laptop:
192.168.1.5

Phone:
192.168.1.10
```

These cannot be accessed directly from internet.

---

# Real Life Example

Home network:

```text
Internet
    |
Public IP
49.37.x.x
    |
Router
   / \
Laptop  Phone

192.168.1.5
192.168.1.6
```

Outside world sees only:

```text
49.37.x.x
```

Router manages internal devices.

---

# NAT (Network Address Translation)

Extremely important.

Since IPv4 addresses are limited:

Instead of giving every device a public IP:

```text
Many Private IPs
        ↓
One Public IP
```

Router translates traffic.

Example:

```text
Laptop
192.168.1.5
        ↓
Router
        ↓
49.37.x.x
```

Google sees:

```text
49.37.x.x
```

not the private IP.

---

# CIDR Notation

Very common in HLD interviews.

Example:

```text
192.168.1.0/24
```

Read as:

```text
Network Address
+
Subnet Mask
```

---

# What Does /24 Mean?

```text
192.168.1.0/24
```

means:

```text
24 bits → Network
8 bits → Host
```

Binary:

```text
11111111.11111111.11111111.00000000
```

Subnet Mask:

```text
255.255.255.0
```

---

# Number of Hosts

Formula:

```text
2^(Host Bits)
```

Example:

```text
/24

Host Bits = 8

2^8 = 256
```

Usable:

```text
254
```

because:

```text
Network Address
Broadcast Address
```

are reserved.

---

# Quick CIDR Table

| CIDR | Hosts |
| ---- | ----- |
| /32  | 1     |
| /30  | 2     |
| /29  | 6     |
| /28  | 14    |
| /27  | 30    |
| /26  | 62    |
| /25  | 126   |
| /24  | 254   |
| /23  | 510   |
| /22  | 1022  |
| /16  | 65534 |

Memorize:

```text
/24 = 254 hosts
/16 = 65K hosts
```

Interview favorite.

---

# Subnetting

Subnetting divides large networks into smaller networks.

Example:

Instead of:

```text
10.0.0.0/16
```

create:

```text
10.0.1.0/24

10.0.2.0/24

10.0.3.0/24
```

Benefits:

```text
Security
Isolation
Routing efficiency
Management
```

---

# VPC Example (AWS)

Common HLD question.

Suppose:

```text
VPC
10.0.0.0/16
```

Subnets:

```text
Public Subnet:
10.0.1.0/24

Private Subnet:
10.0.2.0/24
```

Architecture:

```text
Internet
    |
Load Balancer
10.0.1.x
    |
App Servers
10.0.2.x
    |
Database
10.0.3.x
```

Interviewers love this.

---

# Loopback Address

Special IP:

```text
127.0.0.1
```

Means:

```text
My own machine
```

Also called:

```text
localhost
```

Example:

```bash
http://localhost:8080
```

Used for development.

---

# DNS and IP

Humans remember:

```text
google.com
```

Computers use:

```text
142.x.x.x
```

DNS converts:

```text
google.com
      ↓
IP Address
```

before connection starts.

---

# Load Balancer & IP

Example:

```text
Client
   |
Load Balancer
52.x.x.x
   |
-------------
|     |     |
S1    S2    S3
10.x 10.x 10.x
```

Public IP belongs to LB.

Servers usually use private IPs.

---

# Multi-Region HLD Example

```text
India Users
     |
  DNS
     |
--------------------
|                  |
Mumbai Region      Singapore Region
13.x.x.x           18.x.x.x
```

DNS routes users to nearest IP.

---

# Packet Routing Using IP

Suppose:

```text
Source:
192.168.1.5

Destination:
142.250.183.14
```

Router checks:

```text
Destination IP
```

and forwards packet through internet until it reaches destination.

Think:

```text
Courier Service

Address on parcel
        ↓
Routers read address
        ↓
Deliver parcel
```

---

# What Interviewers Usually Ask

### Q1: Difference between Public and Private IP?

```text
Public:
Accessible over internet

Private:
Only inside local network
```

---

### Q2: Why NAT?

```text
IPv4 shortage

Many devices
share one public IP
```

---

### Q3: What is CIDR?

```text
IP range notation

Example:
10.0.0.0/24
```

---

### Q4: Why subnetting?

```text
Security
Isolation
Scalability
Efficient routing
```

---

### Q5: Why are servers inside private subnet?

```text
Security

Internet
    ↓
Load Balancer
    ↓
Private App Servers
    ↓
Private Database
```

Attack surface reduced.

---

# 1-Minute HLD Revision

```text
IP Address = Unique network identifier

IPv4 = 32 bits
IPv6 = 128 bits

Public IP:
Accessible from internet

Private IP:
10.x.x.x
172.16-31.x.x
192.168.x.x

NAT:
Many private IPs → one public IP

CIDR:
10.0.0.0/24

/24 = 254 hosts

Subnetting:
Split large network into smaller networks

Loopback:
127.0.0.1

DNS:
Domain → IP

Load Balancer usually has public IP
Servers and DBs usually have private IPs
```

For HLD interviews, the most important networking chain to understand is:

```text
User
  ↓
DNS
  ↓
Public IP (Load Balancer)
  ↓
Private IP (App Servers)
  ↓
Private IP (Database)
```

This exact flow appears in many system design discussions.
