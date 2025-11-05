
# 🧩 **Lab 13 – PostgreSQL Performance Tuning, Indexing & Monitoring (Hands-on Lab)**

---

## 🎯 **Objectives**

By the end of this lab, learners will:

* Interpret **query execution plans** using `EXPLAIN` and `ANALYZE`.
* Understand **Sequential Scan vs Index Scan** behavior.
* Use **VACUUM**, **ANALYZE**, and **Autovacuum** to maintain performance.
* Design **indexing strategies** for high-volume workloads.
* Apply **table partitioning** to handle large datasets.
* Monitor active queries, locks, and performance metrics with **pg_stat_activity**.

---

## 🧠 **Concept Overview**

| Area                 | Description                                                                                                               |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **Execution Plans**  | PostgreSQL’s optimizer chooses how to execute a query; `EXPLAIN` shows the plan, `EXPLAIN ANALYZE` executes and times it. |
| **Sequential Scan**  | Reads the entire table – efficient for small tables, expensive for large ones.                                            |
| **Index Scan**       | Uses an index to fetch matching rows directly.                                                                            |
| **VACUUM / ANALYZE** | Reclaims storage and updates statistics for the planner.                                                                  |
| **Autovacuum**       | Automatic background maintenance for tables.                                                                              |
| **Partitioning**     | Splits large tables into smaller child tables for faster access.                                                          |
| **pg_stat_activity** | View current backend activity, query text, and locks.                                                                     |

---

## 🧰 **Setup**

Continue with your running container or start a new one:

```bash
docker exec -it postgres-container bash
psql -U postgres
```

Create and connect to a new database:

```sql
CREATE DATABASE performancelab;
\c performancelab;
```

---

# 🧪 **Lab Exercises**

---

## 🔹 **Step 1 – Create Sample Table and Populate Data**

```sql
CREATE TABLE sales (
  id SERIAL PRIMARY KEY,
  product TEXT,
  category TEXT,
  quantity INT,
  price NUMERIC(10,2),
  sale_date DATE
);

-- Insert 1 million rows (simulated for real test)
INSERT INTO sales (product, category, quantity, price, sale_date)
SELECT
  'Product_' || (random()*100)::INT,
  CASE WHEN random() < 0.5 THEN 'Electronics' ELSE 'Clothing' END,
  (random()*10)::INT + 1,
  (random()*5000)::NUMERIC(10,2),
  NOW() - (random()*365)::INT
FROM generate_series(1,1000000);
```

✅ **Observation:** A large table is created to simulate heavy workload.

---

## 🔹 **Step 2 – View Execution Plans (EXPLAIN / ANALYZE)**

```sql
EXPLAIN SELECT * FROM sales WHERE category='Electronics';

EXPLAIN ANALYZE SELECT * FROM sales WHERE category='Electronics';
```

✅ **Observation:**

* Output shows `Seq Scan on sales` because no index exists.
* `EXPLAIN ANALYZE` includes actual execution time.

---

## 🔹 **Step 3 – Create Index and Compare**

```sql
CREATE INDEX idx_sales_category ON sales(category);

EXPLAIN ANALYZE SELECT * FROM sales WHERE category='Electronics';
```

✅ **Observation:**
Now you should see `Index Scan using idx_sales_category` with lower execution time.

---

## 🔹 **Step 4 – Sequential vs Index Scan Comparison**

```sql
SET enable_seqscan = off;    -- Force index scan
EXPLAIN ANALYZE SELECT * FROM sales WHERE category='Clothing';

SET enable_seqscan = on;     -- Restore default
EXPLAIN ANALYZE SELECT * FROM sales WHERE category='Clothing';
```

✅ **Observation:**
Disabling sequential scans forces the planner to use the index even if it’s not optimal.

---

## 🔹 **Step 5 – VACUUM and ANALYZE**

Simulate updates and cleanup:

```sql
UPDATE sales SET price = price * 1.05 WHERE category='Electronics';

VACUUM sales;     -- Reclaim space
ANALYZE sales;    -- Refresh statistics
```

✅ **Observation:**
`ANALYZE` helps the optimizer choose better plans; `VACUUM` prevents table bloat.

### Check Autovacuum Status

```sql
SHOW autovacuum;
SELECT relname, last_vacuum, last_autovacuum FROM pg_stat_user_tables;
```

---

## 🔹 **Step 6 – Composite Index and Performance Measurement**

Create a multi-column index for more selective filtering:

```sql
CREATE INDEX idx_sales_category_date ON sales(category, sale_date);

EXPLAIN ANALYZE
SELECT * FROM sales
WHERE category='Electronics' AND sale_date > NOW() - INTERVAL '30 days';
```

✅ **Observation:**
Planner now performs a faster index scan using the composite index.

---

## 🔹 **Step 7 – Partitioning Large Table**

### 1️⃣ Create Parent Table

```sql
CREATE TABLE sales_partitioned (
  id SERIAL,
  product TEXT,
  category TEXT,
  quantity INT,
  price NUMERIC(10,2),
  sale_date DATE
) PARTITION BY RANGE (sale_date);
```

### 2️⃣ Create Monthly Partitions

```sql
CREATE TABLE sales_2025_01 PARTITION OF sales_partitioned
  FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

CREATE TABLE sales_2025_02 PARTITION OF sales_partitioned
  FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');
```

### 3️⃣ Insert Data and Test

```sql
INSERT INTO sales_partitioned (product, category, quantity, price, sale_date)
SELECT product, category, quantity, price, sale_date
FROM sales WHERE sale_date > '2025-01-01';

EXPLAIN ANALYZE
SELECT * FROM sales_partitioned WHERE sale_date BETWEEN '2025-01-15' AND '2025-01-31';
```

✅ **Observation:**
Planner only scans the relevant partition (partition pruning).

---

## 🔹 **Step 8 – Monitoring Active and Blocked Sessions**

```sql
SELECT pid, usename, state, query, wait_event_type, wait_event
FROM pg_stat_activity
WHERE state != 'idle';

-- Monitor locks
SELECT relation::regclass AS table_name, mode, granted, pid
FROM pg_locks
WHERE NOT granted;
```

✅ **Observation:**
These views help detect long-running or blocked queries.

---

## 🔹 **Step 9 – Identify Slow Queries and Optimize**

### 1️⃣ Enable Query Timing Logs

```sql
ALTER SYSTEM SET log_min_duration_statement = 500; -- log queries slower than 500 ms
SELECT pg_reload_conf();
```

### 2️⃣ Run Slow Query Example

```sql
EXPLAIN ANALYZE SELECT * FROM sales ORDER BY price DESC;
```

Add an index to optimize:

```sql
CREATE INDEX idx_sales_price ON sales(price DESC);
EXPLAIN ANALYZE SELECT * FROM sales ORDER BY price DESC;
```

✅ **Observation:**
Execution time decreases, and the plan now uses `Index Scan Backward`.

---

## 🧾 **Summary Table**

| Area                     | Key Command / Concept                          |
| ------------------------ | ---------------------------------------------- |
| Execution Plan           | `EXPLAIN`, `EXPLAIN ANALYZE`                   |
| Sequential vs Index Scan | `enable_seqscan = off`                         |
| Maintenance              | `VACUUM`, `ANALYZE`, `Autovacuum`              |
| Index Strategy           | Single & composite indexes                     |
| Partitioning             | `PARTITION BY RANGE`                           |
| Monitoring               | `pg_stat_activity`, `pg_locks`                 |
| Query Optimization       | `log_min_duration_statement`, `EXPLAIN` tuning |

---

## ✅ **Deliverables**

Each learner should submit:

1. Screenshots of `EXPLAIN ANALYZE` before / after indexing.
2. Output of `pg_stat_activity` showing active sessions.
3. Evidence of `VACUUM`/`ANALYZE` execution.
4. Partition pruning plan screenshot.
5. A short summary on which optimization had the biggest effect.

---

## 🧩 **Practice Challenges**

1. Create a **covering index** (`INCLUDE` clause) and test performance.
2. Build a **BRIN index** on the `sale_date` column and compare to BTREE.
3. Configure a **custom autovacuum threshold** and observe behavior.
4. Use `pg_stat_statements` (if enabled) to identify top slow queries.
5. Simulate **deadlocks** and inspect with `pg_locks`.

---

