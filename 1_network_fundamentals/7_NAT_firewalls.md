# NAT & Firewalls (Important HLD Interview Concepts)

These two topics appear frequently in discussions about:

* VPCs
* AWS/Azure/GCP
* Network Security
* Load Balancers
* Public vs Private Subnets
* Internet Access for Servers
* Microservices

---

# Part 1: NAT (Network Address Translation)

## Why NAT Exists

IPv4 has only about:

```text
2^32 ≈ 4.3 Billion Addresses
```

But today there are:

```text
Phones
Laptops
Servers
IoT Devices
```

far more than available public IPv4 addresses.

Solution:

```text
Many Private IPs
        ↓
One Public IP
```

using NAT.

---

# Real Example

Home Network:

```text
Internet
    |
Public IP
49.205.10.20
    |
Router
  / | \
 /  |  \
PC Phone TV

192.168.1.2
192.168.1.3
192.168.1.4
```

All devices have private IPs.

Outside world only sees:

```text
49.205.10.20
```

---

# What NAT Does

When your laptop sends a request:

```text
Source IP:
192.168.1.2
```

Google cannot route replies to a private IP.

Router performs NAT:

```text
192.168.1.2
      ↓
49.205.10.20
```

Now packet is valid on the internet.

---

# Packet Flow

### Before NAT

```text
Src IP = 192.168.1.2
Dst IP = 142.250.x.x
```

---

### After NAT

```text
Src IP = 49.205.10.20
Dst IP = 142.250.x.x
```

Google sees:

```text
49.205.10.20
```

not your laptop's IP.

---

# NAT Translation Table

Router keeps a mapping table.

Example:

```text
192.168.1.2:5000
        ↓
49.205.10.20:10001

192.168.1.3:5001
        ↓
49.205.10.20:10002
```

This allows many devices to share one public IP.

---

# Types of NAT

---

## 1. Static NAT

One-to-one mapping.

```text
10.0.0.10
      ↓
52.10.20.30
```

Same mapping always.

Common for:

```text
Servers
```

---

## 2. Dynamic NAT

Public IP selected from a pool.

```text
10.0.0.10
      ↓
52.1.1.1

10.0.0.11
      ↓
52.1.1.2
```

---

## 3. PAT (Port Address Translation)

Most common.

Also called:

```text
NAT Overload
```

Many devices share one public IP.

Different ports distinguish traffic.

```text
192.168.1.2:5000
      ↓
49.205.10.20:10001

192.168.1.3:5000
      ↓
49.205.10.20:10002
```

This is what home routers use.

---

# NAT in AWS (HLD Favorite)

Architecture:

```text
Internet
   |
Public Subnet
   |
NAT Gateway
   |
Private Subnet
   |
App Servers
```

---

## Why?

Private servers should not be publicly reachable.

But they still need internet access for:

```text
Software Updates
Docker Images
External APIs
```

---

Flow:

```text
Private Server
      |
NAT Gateway
      |
Internet
```

Internet cannot initiate traffic back directly.

Very secure.

---

# NAT Advantages

### Conserves IPv4 Addresses

```text
1000 Devices
       ↓
1 Public IP
```

---

### Security

Private IPs are hidden.

---

### Easier Network Management

---

# NAT Limitations

Breaks true end-to-end communication.

Sometimes causes issues for:

```text
VoIP
Gaming
Peer-to-Peer
```

---

# Part 2: Firewall

A firewall is a security system that decides:

```text
Allow Traffic?
or
Block Traffic?
```

Think of it as a security guard.

---

# Real Life Analogy

Office Building:

```text
Security Guard
```

Checks:

```text
Who are you?
Where are you going?
```

Allows or denies entry.

Firewall does the same for network traffic.

---

# Basic Firewall Flow

```text
Internet
    |
 Firewall
    |
 Servers
```

Every packet is inspected.

---

# Example Rules

Allow:

```text
HTTPS (443)
```

Block:

```text
Telnet (23)
```

Rule:

```text
ALLOW TCP 443
DENY TCP 23
```

---

# Firewall Decision Process

Packet arrives:

```text
Src IP
Dst IP
Port
Protocol
```

Firewall checks rules.

Example:

```text
Source: 8.8.8.8
Destination: App Server
Port: 443
```

Rule:

```text
ALLOW
```

Packet passes.

---

# Stateless Firewall

Examines each packet independently.

```text
Packet
 ↓
Check Rules
 ↓
Allow/Block
```

No memory of previous packets.

Fast but less intelligent.

---

# Stateful Firewall

Tracks connections.

Example:

```text
Client
  |
TCP Connection
  |
Server
```

Firewall remembers:

```text
Connection already established
```

and allows response traffic automatically.

Much more common today.

---

# Network Firewall vs Host Firewall

## Network Firewall

Protects many machines.

Example:

```text
Internet
    |
Firewall
    |
Servers
```

Examples:

```text
AWS Security Appliances
Cisco ASA
Palo Alto
```

---

## Host Firewall

Runs on a machine.

Examples:

```text
Windows Firewall
iptables
ufw
```

Protects a single server.

---

# AWS Security Group

Very common HLD topic.

Security Group acts like a virtual firewall.

Example:

```text
App Server
```

Allow:

```text
TCP 443
TCP 80
```

Block everything else.

---

Rule Example:

```text
Inbound:
443 from 0.0.0.0/0

Outbound:
All
```

---

# Security Group Example

```text
Internet
    |
Load Balancer
    |
App Server
```

Rules:

### LB

```text
Allow 443 from Internet
```

### App Server

```text
Allow traffic only from LB
```

### Database

```text
Allow traffic only from App Server
```

---

# Firewall in HLD Architecture

```text
Internet
    |
Firewall
    |
Load Balancer
    |
App Servers
    |
Database
```

Security layers increase as we move inward.

---

# Common HLD Design Pattern

```text
Internet
   |
Public Load Balancer
   |
Public Subnet
   |
App Servers
(Private Subnet)
   |
Database
(Private Subnet)
```

Firewall rules:

```text
Internet -> LB
Allowed

Internet -> App Server
Blocked

Internet -> DB
Blocked
```

---

# NAT + Firewall Together

Typical architecture:

```text
Internet
   |
Firewall
   |
NAT Gateway
   |
Private App Servers
   |
Database
```

Firewall:

```text
Controls traffic
```

NAT:

```text
Provides outbound internet access
```

---

# Interview Questions

### What is NAT?

```text
Private IP ↔ Public IP translation
```

---

### Why NAT?

```text
IPv4 shortage
Security
```

---

### What is NAT Gateway?

Allows private servers to access the internet without being publicly reachable.

---

### What is a Firewall?

Filters traffic using security rules.

---

### Stateful vs Stateless Firewall?

```text
Stateless:
No connection tracking

Stateful:
Tracks connections
```

---

### Why Put DB in Private Subnet?

```text
Security
```

Database should never be publicly exposed.

---

### Why Allow Only LB to Access App Servers?

```text
Reduce attack surface
```

Only trusted traffic reaches backend.

---

# Typical Interview Architecture

```text
User
  |
DNS
  |
Load Balancer (Public)
  |
Firewall Rules
  |
App Servers (Private)
  |
Redis
  |
Database (Private)
```

---

# 1-Minute Revision

```text
NAT:
Private IP → Public IP

Types:
Static NAT
Dynamic NAT
PAT (Most Common)

Purpose:
Save IPv4 addresses
Hide private IPs

Firewall:
Allow/Block traffic

Checks:
IP
Port
Protocol

Types:
Stateless
Stateful

AWS:
Security Group = Firewall

Best Practice:
Internet
 ↓
Load Balancer
 ↓
App Servers (Private)
 ↓
Database (Private)

NAT:
Outbound Internet Access

Firewall:
Traffic Control
```

### Memory Line

```text
NAT changes addresses.
Firewall controls access.

NAT answers:
"How do I reach the internet?"

Firewall answers:
"Should this traffic be allowed?"
```
