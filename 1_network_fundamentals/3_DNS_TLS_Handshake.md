# DNS (Domain Name System) – HLD Interview Deep Dive

DNS is one of the most important networking concepts in System Design.

Whenever a user opens:

```text
google.com
amazon.com
netflix.com
```

the browser must first find the server's IP address.

DNS is the system that converts:

```text
Domain Name
      ↓
IP Address
```

Example:

```text
google.com
      ↓
142.250.xxx.xxx
```

Without DNS, users would need to remember IP addresses instead of domain names.

---

# Why DNS Exists

Humans remember:

```text
amazon.com
youtube.com
netflix.com
```

Computers communicate using:

```text
44.212.10.20
142.250.183.14
```

DNS acts like the internet's phonebook.

---

# Real Request Flow

Suppose user enters:

```text
https://www.google.com
```

The browser cannot directly connect.

First:

```text
www.google.com
       ↓
DNS Lookup
       ↓
142.250.xxx.xxx
       ↓
TCP Connection
       ↓
HTTPS Request
```

DNS always happens before the actual request.

---

# DNS Hierarchy

DNS is a distributed hierarchical system.

```text
                    .
                   Root
                    |
       ------------------------
       |          |           |
      .com       .org       .net
       |
   google.com
       |
   www.google.com
```

No single server stores the entire internet's DNS records.

---

# DNS Components

## 1. Client

Usually:

```text
Browser
Mobile App
Operating System
```

Makes DNS query.

---

## 2. Recursive Resolver

Usually provided by:

```text
ISP
Google DNS (8.8.8.8)
Cloudflare (1.1.1.1)
```

Its job:

```text
Find the answer for the client.
```

---

## 3. Root Name Server

Knows where TLD servers are.

Example:

```text
Where is .com?
```

---

## 4. TLD Server

TLD = Top Level Domain

Examples:

```text
.com
.org
.net
.in
```

Knows where domain servers are.

Example:

```text
Where is google.com?
```

---

## 5. Authoritative DNS Server

Stores actual records.

Example:

```text
google.com → 142.250.xxx.xxx
```

Final source of truth.

---

# Complete DNS Lookup

Suppose user requests:

```text
www.google.com
```

---

### Step 1

Browser cache checked.

```text
Found?
   Yes → Return IP
   No  → Continue
```

---

### Step 2

OS cache checked.

```text
Found?
   Yes → Return IP
   No  → Continue
```

---

### Step 3

Recursive Resolver queried.

Example:

```text
8.8.8.8
```

---

### Step 4

Resolver asks Root Server:

```text
Where is .com?
```

Root replies:

```text
Ask .com TLD server
```

---

### Step 5

Resolver asks TLD Server:

```text
Where is google.com?
```

Reply:

```text
Ask Google's authoritative DNS
```

---

### Step 6

Resolver asks Authoritative Server:

```text
What is IP of www.google.com?
```

Reply:

```text
142.250.xxx.xxx
```

---

### Step 7

Resolver returns answer to browser.

Browser now connects to that IP.

---

# Visualization

```text
User
 |
Browser Cache
 |
OS Cache
 |
Resolver (8.8.8.8)
 |
Root DNS
 |
.com TLD
 |
Google Authoritative DNS
 |
IP Returned
 |
Browser Connects
```

---

# DNS Caching

Very important for interviews.

Without caching:

Every request would hit:

```text
Root
TLD
Authoritative
```

which would be extremely slow.

Instead:

```text
Browser Cache
OS Cache
Resolver Cache
```

store results.

---

# TTL (Time To Live)

Every DNS record has TTL.

Example:

```text
google.com → 142.250.xxx.xxx

TTL = 300 sec
```

Meaning:

```text
Cache for 5 minutes
```

After expiry:

```text
Lookup again
```

---

# DNS Record Types

Interview favorite.

---

## A Record

Maps:

```text
Domain → IPv4
```

Example:

```text
google.com
    ↓
142.250.183.14
```

---

## AAAA Record

Maps:

```text
Domain → IPv6
```

Example:

```text
google.com
    ↓
2001:4860:4860::8888
```

---

## CNAME

Alias record.

Example:

```text
api.company.com
       ↓
backend.company.com
```

Useful for flexibility.

---

## MX Record

Mail server.

Example:

```text
gmail.com
      ↓
Mail Server
```

Used for email routing.

---

## NS Record

Name Server record.

Specifies:

```text
Who is authoritative?
```

---

# DNS and Load Balancing

Very common HLD topic.

---

Suppose Netflix has servers:

```text
US
Europe
India
```

DNS can return different IPs.

```text
India User
      ↓
Indian Server IP

US User
      ↓
US Server IP
```

This is DNS-based load balancing.

---

# Geo DNS

User location determines response.

Example:

```text
Mumbai User
       ↓
Mumbai Server

Singapore User
       ↓
Singapore Server
```

Benefits:

```text
Lower latency
Better user experience
```

---

# DNS and CDN

CDNs heavily rely on DNS.

Example:

```text
cdn.netflix.com
```

DNS returns nearest edge server.

```text
User
   |
DNS
   |
Nearest CDN Node
```

Instead of hitting origin server.

---

# DNS Round Robin

Simple load balancing technique.

Suppose:

```text
api.company.com
```

Maps to:

```text
10.1.1.1
10.1.1.2
10.1.1.3
```

DNS may return:

```text
Request 1 → 10.1.1.1
Request 2 → 10.1.1.2
Request 3 → 10.1.1.3
```

Traffic gets distributed.

---

# DNS Failover

Suppose:

```text
Mumbai Server Down
```

DNS can return:

```text
Singapore Server IP
```

instead.

Used in:

```text
Disaster Recovery
Multi-region Systems
```

---

# DNS in System Design Architecture

Typical flow:

```text
User
  |
DNS
  |
Load Balancer
  |
App Servers
  |
Database
```

Example:

```text
amazon.com
     |
DNS
     |
52.x.x.x
     |
Load Balancer
     |
Application Servers
```

---

# Common HLD Interview Questions

## Q1: Why DNS caching?

Answer:

```text
Reduce latency
Reduce DNS traffic
Improve performance
```

---

## Q2: What is TTL?

```text
How long DNS result remains cached.
```

---

## Q3: Why multiple DNS servers?

```text
High Availability
Fault Tolerance
Scalability
```

---

## Q4: How does Netflix send users to nearest server?

```text
Geo DNS
CDN
```

---

## Q5: Can DNS be used for load balancing?

Yes.

```text
DNS Round Robin
Geo Routing
Weighted Routing
```

---

## Q6: Is DNS over TCP or UDP?

Mostly:

```text
UDP Port 53
```

because it's fast.

Sometimes:

```text
TCP Port 53
```

for large responses or zone transfers.

---

# Complete Flow of Opening Google

```text
1. User enters google.com

2. Browser checks DNS cache

3. OS checks DNS cache

4. Resolver queried

5. Root DNS

6. .com TLD

7. Google Authoritative DNS

8. IP returned

9. TCP Handshake

10. TLS Handshake

11. HTTPS Request

12. Google Response
```

---

# 1-Minute HLD Revision

```text
DNS = Domain Name System

Purpose:
Domain → IP Address

Hierarchy:
Root
 ↓
TLD (.com)
 ↓
Authoritative DNS

Important Records:
A      → IPv4
AAAA   → IPv6
CNAME  → Alias
MX     → Mail
NS     → Name Server

Caching:
Browser Cache
OS Cache
Resolver Cache

TTL:
How long cache remains valid

Port:
53 UDP (mostly)
53 TCP (sometimes)

Used For:
Load Balancing
Geo Routing
CDNs
Failover
Multi-region Systems
```

### Interview Memory Line

```text
User
 ↓
DNS
 ↓
Load Balancer
 ↓
Application Servers
 ↓
Database

DNS is the first network component involved in almost every web request.
```
