### What is Indexing in Databases?

**Indexing** is a technique used by databases to **find data quickly without scanning the entire table**.

Think of it like the **index page of a book**.

---

### Without Index

Suppose you have a `Users` table with **10 million rows**.

```sql
SELECT * FROM Users
WHERE email = 'jay@example.com';
```

If there is **no index** on `email`:

Database checks every row one by one.

```text
Row 1  -> No
Row 2  -> No
Row 3  -> No
...
Row 10,000,000 -> Found
```

This is called a **Full Table Scan**.

Time Complexity:

```text
O(N)
```

Very slow for large datasets.

---

### With Index

Create an index:

```sql
CREATE INDEX idx_email
ON Users(email);
```

Database creates a special data structure (usually a **B+ Tree**).

```text
                 M
              /     \
            G         T
          /  \      /   \
        A-F H-L   N-S   U-Z
```

Now the database directly navigates to the correct location.

Time Complexity:

```text
O(log N)
```

Much faster.

---

## Real Life Example

### Without Index

Imagine finding:

```text
Employee ID = 93245
```

in a file cabinet containing 1 million employee records.

You must open every file until you find it.

---

### With Index

There is a separate book:

```text
93245 -> Drawer 17, File 43
```

You directly go to the file.

This lookup book is the **index**.

---

# How Databases Store Indexes

Most SQL databases use:

```text
B+ Trees
```

Examples:

* MySQL InnoDB
* PostgreSQL
* SQL Server
* Oracle

MongoDB also uses B-Tree-like indexes (WiredTiger).

---

## Table vs Index

### Actual Table

```text
ID   Name    Age
----------------
1    John    25
2    Mike    30
3    Sam     22
```

### Index on Age

```text
22 -> Row 3
25 -> Row 1
30 -> Row 2
```

The index stores:

```text
Indexed Value -> Pointer to Row
```

not the entire row.

---

# Clustered Index

In a clustered index, the table data itself is stored in index order.

Example:

```text
PRIMARY KEY(id)
```

Data on disk:

```text
1 -> John
2 -> Mike
3 -> Sam
4 -> Alex
```

Rows are physically ordered.

### Benefits

Very fast:

```sql
SELECT * FROM Users
WHERE id = 100;
```

and range queries:

```sql
SELECT *
FROM Users
WHERE id BETWEEN 100 AND 200;
```

---

# Non-Clustered Index

Separate structure from actual data.

```text
Age Index

22 -> Pointer Row3
25 -> Pointer Row1
30 -> Pointer Row2
```

Actual table remains elsewhere.

When querying:

```sql
SELECT * FROM Users
WHERE age = 25;
```

Database:

```text
Age Index
   ↓
Find Row Pointer
   ↓
Go to Table
   ↓
Fetch Data
```

---

# Composite Index

Index on multiple columns.

```sql
CREATE INDEX idx_user
ON Users(country, city);
```

Stored like:

```text
(India, Mumbai)
(India, Pune)
(USA, Boston)
(USA, NYC)
```

Good for:

```sql
WHERE country='India'
```

and

```sql
WHERE country='India'
AND city='Mumbai'
```

Not very useful for:

```sql
WHERE city='Mumbai'
```

because indexes follow the **Leftmost Prefix Rule**.

---

# Covering Index

Suppose:

```sql
CREATE INDEX idx_user
ON Users(name, age);
```

Query:

```sql
SELECT name, age
FROM Users
WHERE name='John';
```

Everything needed is already inside the index.

Database doesn't access the table.

```text
Index Only Scan
```

Very fast.

---

# Why Not Index Every Column?

Indexes speed up reads but slow down writes.

When inserting:

```sql
INSERT INTO Users ...
```

Database must:

1. Insert row
2. Update Index 1
3. Update Index 2
4. Update Index 3

More indexes ⇒ slower writes.

---

# Example

### No Index

```sql
SELECT *
FROM Orders
WHERE order_id = 5000000;
```

For 10 million rows:

```text
Read ~10 million rows
```

---

### With Index

```sql
CREATE INDEX idx_order
ON Orders(order_id);
```

Database:

```text
Root
 ↓
Internal Node
 ↓
Leaf Node
 ↓
Row
```

Reads only a few pages.

---

# Indexing in SQL vs MongoDB

### MySQL/PostgreSQL

```sql
CREATE INDEX idx_email
ON Users(email);
```

Uses:

```text
B+ Tree
```

---

### MongoDB

```javascript
db.users.createIndex({
   email: 1
})
```

Uses WiredTiger's B-Tree based indexes.

---

# HLD Interview One-Liner

**Index = a separate data structure (usually a B+ Tree) that stores column values and pointers to rows, allowing the database to locate records in O(log N) time instead of scanning the entire table in O(N) time.**

### Quick Revision

```text
Without Index
-------------
Lookup = Full Table Scan
Complexity = O(N)

With Index
----------
Lookup = B+ Tree Search
Complexity = O(log N)

Types
-----
Clustered Index
Non-Clustered Index
Composite Index
Covering Index

Pros
-----
Fast Reads
Fast Searches
Fast Range Queries

Cons
-----
Extra Storage
Slower Inserts
Slower Updates
Slower Deletes
```

For HLD interviews, the most important connection is:

```text
Indexing
   ↓
Implemented using B+ Trees
   ↓
Stored on Disk Efficiently
   ↓
Works together with WAL for durability
```

That's why **B+ Trees + Indexing + WAL** are often discussed together when explaining database internals.
