
---

# 1. Geohash

## Problem

Suppose Uber wants:

> Find all drivers within 5 km.

You cannot scan millions of drivers every time.

---

## Idea

Convert latitude and longitude into a string.

Example:

Mumbai

```
19.0760,72.8777
```

becomes

```
te7ud
```

Nearby locations produce similar prefixes.

```
te7ud
te7ue
te7uf
```

---

## Real World Example

Uber Driver Search

```
Driver A → te7ud
Driver B → te7ue
Driver C → te7uf
```

To find nearby drivers:

```
SELECT * WHERE geohash LIKE 'te7u%'
```

Instead of checking every driver.

---

## Advantages

### Fast Search

```
O(log n)
```

instead of

```
O(n)
```

---

### Easy Database Indexing

Store as string.

```
te7ud
```

and index it.

---

## Interview Usage

Used in:

* Uber
* Ola
* Swiggy
* Zomato
* Nearby Restaurants
* Delivery Systems

---

# 2. Quad Trees

## Problem

Suppose map contains:

```
100 Million Points
```

Need:

```
Find all restaurants inside area
```

---

## Idea

Keep dividing map into 4 pieces.

```
World
 ├─ NW
 ├─ NE
 ├─ SW
 └─ SE
```

If a region becomes crowded:

```
divide again
```

---

Visualization

```
+-------+
| A | B |
|---|---|
| C | D |
+-------+
```

Each box can further split.

---

## Search

Want restaurants in:

```
Mumbai Area
```

Only traverse relevant boxes.

---

## Interview Usage

Google Maps

Ride Sharing

Gaming

GIS Systems

---

## Why Better?

Instead of:

```
Search 100M points
```

Search:

```
Few relevant boxes
```

---

# 3. R-Trees

## Problem

Quad Trees work well for points.

But maps contain:

* Roads
* Buildings
* Parks
* Regions

These are rectangles/polygons.

---

## Idea

Store Bounding Boxes.

Example

```
Mall
```

becomes

```
+------+
| Mall |
+------+
```

Many nearby boxes grouped.

---

Structure

```
Root
 ├── City A
 ├── City B
 └── City C
```

Each node stores bounding rectangles.

---

## Interview Usage

Google Maps

PostGIS

GIS databases

Location Search

---

## Difference from Quad Tree

Quad Tree:

```
Divides Space
```

R Tree:

```
Groups Objects
```

---

# 4. Skip Lists

Alternative to balanced trees.

---

Problem

Need fast:

```
Insert
Delete
Search
```

---

Normal Linked List

```
1 → 2 → 3 → 4 → 5
```

Search:

```
O(n)
```

---

Skip List

Add shortcuts.

```
1 ------> 5
 \
  ->3
```

Now jumps happen.

---

Search

Instead of:

```
1→2→3→4→5
```

Jump:

```
1→5
```

---

Complexity

Average

```
Search O(log n)
Insert O(log n)
Delete O(log n)
```

---

Interview Usage

Redis Sorted Sets

Apache Cassandra internals

Database indexes

---

Why Engineers Love It

Simpler than Red-Black Trees.

---

# 5. Merkle Trees

Extremely important in distributed systems.

---

## Problem

Two servers have:

```
1 Billion Records
```

Need to know:

```
Which records differ?
```

Don't compare everything.

---

## Idea

Hash data recursively.

```
        Root Hash
       /        \
    H1          H2
   / \         / \
 D1 D2      D3 D4
```

---

If Root Hash matches:

```
Everything matches
```

Done.

---

If Root Hash differs:

Check children.

```
H1
H2
```

Only traverse differing branch.

---

## Example

Database Replication

```
Primary
Replica
```

Need synchronization.

Merkle Tree quickly identifies mismatch.

---

## Interview Usage

Bitcoin

Apache Cassandra

Distributed Databases

Data Replication

---

# 6. HyperLogLog (HLL)

Very common HLD topic.

---

## Problem

Count:

```
Unique Users Today
```

For:

```
500 Million Events
```

---

Naive

Store all IDs.

```
Set<UserId>
```

Memory huge.

---

HyperLogLog

Uses probabilistic math.

Stores tiny metadata.

---

Example

Instead of storing

```
1
5
20
100
200
...
```

Stores patterns.

Estimates:

```
~100M unique users
```

---

Accuracy

Typically

```
1-2% error
```

Memory

```
KBs instead of GBs
```

---

Interview Usage

Analytics

DAU

MAU

Unique Visitors

Ad Systems

---

Question

"How would you count daily active users?"

Answer:

```
HyperLogLog
```

---

# 7. Count-Min Sketch

Another probabilistic data structure.

---

## Problem

Find:

```
Most searched products
```

from billions of searches.

---

Naive

```
Map<Product,Count>
```

Huge memory.

---

Count-Min Sketch

Uses multiple hash tables.

```
Hash1
Hash2
Hash3
```

Update counts.

---

Query

```
How many times Product X appeared?
```

Return estimate.

---

Memory

Very small.

---

Accuracy

May slightly overestimate.

Never underestimates.

---

Interview Usage

Trending Hashtags

Top Searches

Heavy Hitters

Fraud Detection

Analytics

---

# HLD Interview Cheat Sheet

### Location-Based Systems

Study:

* Geohash
* Quad Trees

Examples:

* Uber
* Ola
* Swiggy
* Zomato

---

### Distributed Databases

Study:

* Merkle Trees
* Skip Lists

Examples:

* Cassandra
* Replication

---

### Analytics Systems

Study:

* HyperLogLog
* Count-Min Sketch

Examples:

* DAU/MAU counting
* Trending products
* Search analytics

---
