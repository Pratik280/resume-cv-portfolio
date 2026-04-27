# 🧠 What is Database Indexing?

An **index** is a data structure that helps the database find rows faster **without scanning the entire table**.

Think of it like a book index:

* Without index → you read every page (full table scan)
* With index → jump directly to the page (fast lookup)

---

# ⚙️ How it works (under the hood)

Most databases (like MySQL, PostgreSQL, Oracle Database) use a structure called:

👉 **B-Tree (or B+ Tree)**

### Why B-Tree?

* Sorted structure
* Logarithmic search → `O(log n)`
* Efficient for range queries

---

# 🔍 Without vs With Index

### Without Index

```sql
SELECT * FROM users WHERE email = 'test@example.com';
```

➡ DB scans every row (slow for large tables)

### With Index

```sql
CREATE INDEX idx_email ON users(email);
```

➡ DB jumps directly using index (fast)

---

# 📦 Types of Indexes (Important for Interviews)

## 1. Single Column Index

```sql
CREATE INDEX idx_name ON users(name);
```

* Speeds up queries on one column

---

## 2. Composite Index (Multi-column)

```sql
CREATE INDEX idx_name_age ON users(name, age);
```

⚠️ Order matters:

* Works for `(name)` ✅
* Works for `(name, age)` ✅
* Does NOT work for `(age)` ❌

👉 Called **Leftmost Prefix Rule**

---

## 3. Unique Index

```sql
CREATE UNIQUE INDEX idx_email ON users(email);
```

* Ensures no duplicates
* Used for constraints like email, username

---

## 4. Primary Key Index

```sql
PRIMARY KEY (id)
```

* Automatically indexed
* Usually clustered (in many DBs)

---

## 5. Full-Text Index

* Used for searching text

```sql
MATCH(content) AGAINST('search')
```

---

## 6. Hash Index (less common)

* O(1) lookup
* Only works for exact matches (no range queries)

---

# ⚡ When Index is Used

DB uses index when:

* `WHERE column = value`
* `JOIN` conditions
* `ORDER BY`
* `GROUP BY`

---

# 🚫 When Index is NOT Used

Even if index exists, DB may ignore it:

* Small table (full scan is faster)
* Using functions:

```sql
WHERE LOWER(name) = 'pratik'  -- index not used
```

* Leading wildcard:

```sql
WHERE name LIKE '%abc'  -- no index
```

---

# 💸 Cost of Index (Very Important)

Index is not free:

### 1. Slows down writes

* INSERT
* UPDATE
* DELETE

Because index must also be updated

---

### 2. Takes extra space

* Stored separately

---

👉 Interview line:

> “Indexes improve read performance at the cost of write performance and storage.”

---

# 🧩 Clustered vs Non-Clustered Index

## Clustered Index

* Data is physically sorted
* Only **one per table**

Example:

```sql
PRIMARY KEY
```

---

## Non-Clustered Index

* Separate structure pointing to data
* Can have multiple

---

# 📊 Real Example

Table: `orders`

Query:

```sql
SELECT * FROM orders WHERE user_id = 101;
```

👉 Best index:

```sql
CREATE INDEX idx_user_id ON orders(user_id);
```

---

# 🎯 How to Decide What to Index

Index columns:

* Frequently used in `WHERE`
* Used in `JOIN`
* Used in `ORDER BY`

Avoid indexing:

* Very small tables
* Columns with low uniqueness (like gender)

---

# 🚀 Advanced Concepts (for strong interviews)

## Covering Index

Index contains all required columns:

```sql
CREATE INDEX idx_cover ON users(name, age);
```

Query:

```sql
SELECT name, age FROM users WHERE name = 'A';
```

➡ DB doesn’t even touch table (super fast)

---

## Index Selectivity

* High uniqueness = good index
* Example:

  * email ✅
  * gender ❌

---

## Index Scan vs Index Seek

* **Index Seek** → fast (direct lookup)
* **Index Scan** → slower (scans index)

---

# 🧾 Perfect Interview Answer (Short)

> “Database indexing is a technique to improve query performance by using data structures like B-Trees to quickly locate rows instead of scanning entire tables. Indexes are created on columns frequently used in WHERE, JOIN, and ORDER BY clauses. While they significantly improve read performance, they add overhead to write operations and consume extra storage. Choosing the right index type and columns is crucial for optimal performance.”

---