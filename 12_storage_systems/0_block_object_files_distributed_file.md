# Storage Systems for HLD Interviews

Storage is one of the most important topics in System Design interviews because every large-scale system needs a place to store data.

A simple way to think about storage:

| Storage Type            | Think Like                                  |
| ----------------------- | ------------------------------------------- |
| Block Storage           | Raw hard disk                               |
| File Storage            | Shared folder                               |
| Object Storage          | Huge internet-scale storage bucket          |
| Distributed File System | Many servers acting like one storage system |

---

# 1. Block Storage

## What is it?

Block storage stores data as fixed-size blocks.

The operating system combines these blocks into files.

Think of it as:

```
Hard Disk
 ├── Block 1
 ├── Block 2
 ├── Block 3
 └── Block 4
```

The storage system doesn't know:

* file names
* folders
* images
* videos

It only knows blocks.

---

## Real Life Analogy

Imagine:

```
Warehouse
 ├─ Box 1
 ├─ Box 2
 ├─ Box 3
```

Warehouse only stores boxes.

You decide what is inside.

---

## Examples

### AWS EBS

Elastic Block Store

Used with EC2 machines.

### SSD

### HDD

### SAN Storage

Storage Area Network

---

## Advantages

### Very Fast

Low latency.

Perfect for databases.

### Random Access

Can directly access any block.

---

## Disadvantages

### Not easy to share

Usually attached to one machine.

### Scaling can be difficult

Need larger disks.

---

## HLD Usage

Used for:

* MySQL
* PostgreSQL
* MongoDB
* Cassandra

Because databases need fast random reads/writes.

---

# Interview Line

> Block storage provides low-latency random access and is commonly used by databases.

---

# 2. File Storage

## What is it?

Stores data as:

```
Folders
   ├── file1.txt
   ├── photo.jpg
   └── movie.mp4
```

Storage system understands:

* files
* directories
* permissions

---

## Real Life Analogy

Office filing cabinet.

```
Cabinet
 ├── HR Folder
 ├── Finance Folder
 └── Engineering Folder
```

---

## Examples

### NFS

Network File System

### SMB

Windows File Sharing

### NAS

Network Attached Storage

---

## Access

Many machines can access same files.

```
Server A
      \
       NAS
      /
Server B
```

---

## Advantages

### Easy to Use

Humans understand files.

### Shared Storage

Multiple servers can access.

---

## Disadvantages

### Slower than block storage

Extra metadata management.

### Not ideal for massive internet scale

---

## HLD Usage

Used for:

* shared company drives
* media editing
* application logs
* document storage

---

## Interview Line

> File storage provides a hierarchical file system and is useful when multiple systems need shared access to files.

---

# 3. Object Storage

One of the most important HLD concepts.

---

## What is it?

Stores data as objects.

Each object contains:

```
Object
 ├── Data
 ├── Metadata
 └── Unique ID
```

Instead of folders and blocks.

---

## Example

Store image:

```
profile.jpg
```

Object Storage stores:

```
Object ID: abc123

Data:
  image bytes

Metadata:
  size
  owner
  upload date
```

---

## Real Life Analogy

Huge warehouse.

Every package gets a unique barcode.

You retrieve package using barcode.

Not by folder location.

---

## Examples

### Amazon S3

Most famous.

### Google Cloud Storage

### Azure Blob Storage

### MinIO

---

## Access

Through APIs.

```
PUT Object
GET Object
DELETE Object
```

Not mounted like a normal disk.

---

## Advantages

### Massive Scale

Can store trillions of objects.

### Cheap

Lower cost.

### Extremely Durable

Built for long-term storage.

---

## Disadvantages

### Higher latency

Not good for database storage.

### No random block updates

Usually replace entire object.

---

## HLD Usage

Store:

* images
* videos
* backups
* logs
* user uploads

Example:

Instagram photos → S3

Netflix videos → Object Storage

---

## Interview Line

> Object storage is highly scalable, durable, and cost-effective, making it ideal for storing unstructured data such as images, videos, and backups.

---

# Block vs File vs Object Storage

| Feature  | Block     | File         | Object        |
| -------- | --------- | ------------ | ------------- |
| Unit     | Block     | File         | Object        |
| Speed    | Fastest   | Medium       | Slower        |
| Scale    | Medium    | Medium       | Massive       |
| Metadata | Minimal   | Basic        | Rich          |
| Best For | Databases | Shared Files | Images/Videos |
| Example  | EBS       | NFS          | S3            |

---

# 4. Distributed File Systems

## Problem

Single storage server becomes bottleneck.

```
Users
  |
Storage Server
```

Issues:

* storage full
* server crashes
* limited throughput

---

## Solution

Spread files across many machines.

```
        DFS

  Node1   Node2   Node3
     \      |      /
      \     |     /
        One File System
```

Looks like one file system.

Actually many servers.

---

## Examples

### HDFS

Hadoop Distributed File System

### Google File System (GFS)

### CephFS

### GlusterFS

---

## Example

100 GB file

Split into chunks:

```
Chunk 1 → Server A
Chunk 2 → Server B
Chunk 3 → Server C
```

---

## Advantages

### Horizontal Scaling

Add more machines.

### Fault Tolerance

Machine failures tolerated.

### Parallel Reads

Very high throughput.

---

## HLD Usage

Big Data systems.

```
Hadoop
Spark
Data Lakes
```

---

## Interview Line

> Distributed file systems distribute data across multiple machines to achieve scalability, fault tolerance, and high throughput.

---

# 5. Data Durability

One of the most asked interview concepts.

---

## What is Durability?

Data should survive:

* server crash
* disk crash
* power failure

---

## Example

User uploads photo.

If disk dies after upload:

❌ Photo lost

Bad durability.

---

### Durable System

Photo still exists.

✅ Data survives.

---

# Replication

## What is it?

Store multiple copies.

```
Copy 1 → Server A
Copy 2 → Server B
Copy 3 → Server C
```

---

If:

```
Server B crashes
```

Still available from:

```
Server A
Server C
```

---

# Replication Factor

Number of copies.

```
RF = 3
```

Means:

```
3 copies
```

stored on different machines.

---

# Example

HDFS:

```
Replication Factor = 3
```

```
Chunk
  ├─ Node1
  ├─ Node2
  └─ Node3
```

---

# Why Replication?

### Availability

Server failure doesn't affect users.

### Durability

Data survives hardware failures.

### Faster Reads

Read from nearest replica.

---

# Trade-Off

More replicas:

✅ More durable

❌ More storage cost

---

# Replication vs Backup

## Replication

```
Live copies
```

Purpose:

High availability.

---

## Backup

```
Point-in-time copy
```

Purpose:

Recover deleted/corrupted data.

---

Example:

```
User deletes photo
```

Replication copies deletion too.

Photo gone everywhere.

Need backup.

---

# Interview Line

> Replication improves availability and durability by storing multiple copies of data, whereas backups protect against accidental deletion and corruption.

---

# Quick HLD Revision Sheet

```
BLOCK STORAGE
- Raw blocks
- Fastest
- Database storage
- Example: EBS, SSD

FILE STORAGE
- Files + folders
- Shared access
- Example: NFS, NAS

OBJECT STORAGE
- Objects + metadata
- Massive scale
- Example: S3
- Best for images/videos

DISTRIBUTED FILE SYSTEM
- Many servers = one filesystem
- Example: HDFS, GFS, CephFS
- Scalable and fault tolerant

DURABILITY
- Data survives failures

REPLICATION
- Multiple copies of data
- RF=3 common
- Improves availability + durability

BACKUP
- Point-in-time recovery
- Protects against deletion/corruption
```

### HLD Interview Rule of Thumb

* **Database data** → Block Storage (EBS/SSD)
* **Shared office files** → File Storage (NFS/NAS)
* **Photos, videos, uploads** → Object Storage (S3)
* **Big Data / Hadoop** → Distributed File System (HDFS)
* **Critical data** → Replication + Backups for durability and recovery.
