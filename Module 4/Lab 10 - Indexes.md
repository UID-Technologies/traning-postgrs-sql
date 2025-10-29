
# 🧩 **Lab 10 - Indexes in PostgreSQL**

---

## 🎯 **Objectives**

By the end of this module, learners will:

* Understand what **indexes** are and how they improve query performance.
* Learn the **types of indexes** in PostgreSQL.
* Practice creating, viewing, and deleting indexes.
* Measure the impact of indexes using the **`EXPLAIN ANALYZE`** command.

---

## 🧠 **What is an Index?**

An **Index** is a special data structure that improves the **speed of data retrieval** from a table.
It’s like an index in a book — it helps the database find records faster without scanning the entire table.

### 💡 Analogy:

> Without an index → reading the entire book page by page.
> With an index → jumping directly to the page number you need.

---

## ⚙️ **How Indexes Work**

* PostgreSQL indexes are typically implemented using **B-trees**.
* When you query on an indexed column (e.g., `WHERE email='abc@xyz.com'`), PostgreSQL uses the index to **find rows faster**.
* Indexes make **SELECT** queries faster but can **slow down INSERT, UPDATE, DELETE**, since indexes also need to be updated.

---

## 🧱 **Types of Indexes in PostgreSQL**

| Type                                 | Description                                             | Example Use                        |
| ------------------------------------ | ------------------------------------------------------- | ---------------------------------- |
| **B-Tree (default)**                 | Balanced tree for fast equality/range lookups.          | Most common type, used by default. |
| **Hash Index**                       | Optimized for equality comparisons.                     | `WHERE email = 'abc@xyz.com'`      |
| **GIN (Generalized Inverted Index)** | For text search and JSON fields.                        | Full-text search                   |
| **GiST (Generalized Search Tree)**   | For geometric or full-text search.                      | Range queries, spatial data        |
| **BRIN (Block Range Index)**         | For very large tables, stores ranges instead of values. | Log or time-series data            |
| **Expression Index**                 | Index based on an expression, not just a column.        | `LOWER(email)`                     |
| **Partial Index**                    | Index built on subset of rows using `WHERE` condition.  | Active users only                  |

---

## 🧰 **Setup**

Use your existing schema and employee table, or create a new one for demonstration:

```sql
CREATE SCHEMA IF NOT EXISTS organization AUTHORIZATION postgres;

CREATE TABLE organization.employee (
    emp_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100) UNIQUE,
    base_salary NUMERIC(10,2),
    dept_id INT,
    joining_date DATE
);
```

Insert sample data:

```sql
INSERT INTO organization.employee (first_name, last_name, email, base_salary, dept_id, joining_date)
SELECT
    'Emp' || i, 'Test', 'emp' || i || '@company.com',
    (50000 + (random() * 50000)::int),
    (1 + (random() * 3)::int),
    (CURRENT_DATE - (random() * 1000)::int)
FROM generate_series(1, 10000) AS s(i);
```

✅ **Verify data count**

```sql
SELECT COUNT(*) FROM organization.employee;
```

(should return 10,000 rows)

---

# 🧪 **Lab Exercises – Index Operations**

---

## 🔹 **Step 1 – Create a Basic Index**

### 1.1 Create index on email column

```sql
CREATE INDEX idx_employee_email
ON organization.employee (email);
```

✅ **Check created indexes**

```sql
\d organization.employee;
```

You’ll see:
`Indexes: "employee_pkey" PRIMARY KEY, btree (emp_id), "idx_employee_email" btree (email)`

---

## 🔹 **Step 2 – Measure Query Performance**

### Without Index (drop it first)

```sql
DROP INDEX IF EXISTS idx_employee_email;
```

### Run query and analyze:

```sql
EXPLAIN ANALYZE
SELECT * FROM organization.employee
WHERE email = 'emp5000@company.com';
```

✅ **Observe:** PostgreSQL uses **Seq Scan** (Sequential Scan).

---

### With Index

Recreate index:

```sql
CREATE INDEX idx_employee_email
ON organization.employee (email);
```

Run the same query again:

```sql
EXPLAIN ANALYZE
SELECT * FROM organization.employee
WHERE email = 'emp5000@company.com';
```

✅ **Observation:** PostgreSQL now uses **Index Scan**, much faster.

---

## 🔹 **Step 3 – Multi-Column Index**

If you often query using multiple columns together, create a combined index.

```sql
CREATE INDEX idx_employee_name_city
ON organization.employee (first_name, last_name);
```

Run query:

```sql
EXPLAIN ANALYZE
SELECT * FROM organization.employee
WHERE first_name = 'Emp500' AND last_name = 'Test';
```

✅ **Observation:** Database uses multi-column index if both columns are in the condition.

---

## 🔹 **Step 4 – Unique Index**

A **UNIQUE index** ensures that all values in a column are distinct.

```sql
CREATE UNIQUE INDEX idx_employee_unique_email
ON organization.employee (email);
```

Try inserting a duplicate:

```sql
INSERT INTO organization.employee (first_name, last_name, email, base_salary)
VALUES ('John', 'Doe', 'emp1000@company.com', 70000);
```

❌ Expected: *violates unique constraint*

---

## 🔹 **Step 5 – Partial Index**

Index only active employees (example condition).

```sql
ALTER TABLE organization.employee ADD COLUMN is_active BOOLEAN DEFAULT true;

CREATE INDEX idx_active_employees
ON organization.employee (email)
WHERE is_active = true;
```

✅ **Query uses partial index**

```sql
EXPLAIN ANALYZE
SELECT * FROM organization.employee
WHERE email = 'emp2000@company.com' AND is_active = true;
```

---

## 🔹 **Step 6 – Expression Index**

Create an index on a computed expression (like lowercase email).

```sql
CREATE INDEX idx_employee_lower_email
ON organization.employee (LOWER(email));
```

✅ **Query Example**

```sql
EXPLAIN ANALYZE
SELECT * FROM organization.employee
WHERE LOWER(email) = 'emp3000@company.com';
```

---

## 🔹 **Step 7 – Drop Index**

```sql
DROP INDEX IF EXISTS idx_employee_lower_email;
DROP INDEX IF EXISTS idx_employee_email;
```

✅ **Verify**

```sql
\d organization.employee;
```

(Index entries should be gone.)

---

## 🧮 **Step 8 – View All Indexes**

List all indexes in the database:

```sql
SELECT tablename, indexname, indexdef
FROM pg_indexes
WHERE schemaname = 'organization';
```

---

## 🧾 **Summary Table**

| Type                 | Description               | Syntax Example                                      |
| -------------------- | ------------------------- | --------------------------------------------------- |
| **Basic Index**      | Speeds up lookups         | `CREATE INDEX idx_name ON table(col);`              |
| **Unique Index**     | Prevents duplicates       | `CREATE UNIQUE INDEX idx_u ON table(col);`          |
| **Composite Index**  | Index on multiple columns | `CREATE INDEX idx_m ON table(col1, col2);`          |
| **Partial Index**    | Index on subset of data   | `CREATE INDEX idx_p ON table(col) WHERE condition;` |
| **Expression Index** | Based on expression       | `CREATE INDEX idx_e ON table(LOWER(col));`          |

---

## ⚙️ **Performance Comparison Tips**

| Query Type                         | Index Recommended? | Notes                        |
| ---------------------------------- | ------------------ | ---------------------------- |
| Frequent SELECT with WHERE filters | ✅ Yes              | Great performance gain       |
| Frequent INSERT/UPDATE/DELETE      | ⚠️ Maybe           | Index maintenance overhead   |
| Small tables                       | ❌ No               | Sequential scan often faster |
| Range or equality searches         | ✅ Yes              | Ideal for B-tree indexes     |

---

## ✅ **Deliverables**

Each learner should:

1. Create all types of indexes listed above.
2. Capture `EXPLAIN ANALYZE` results (before/after index).
3. Submit screenshots or `.sql` script showing:

   * Index creation
   * Performance comparison
   * Query plan changes (`Seq Scan` → `Index Scan`)

---

## 💡 **Practice Challenges**

1. Create an index on `base_salary` and verify its usage with a query.
2. Create a partial index on employees who joined after 2022.
3. Create a composite index on `(dept_id, base_salary)` and test queries with both columns.
4. Measure the difference in query time using `EXPLAIN ANALYZE`.
5. Drop all custom indexes and note how performance changes.

---

