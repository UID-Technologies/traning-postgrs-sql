
# 🧩 **Lab 03 – DDL Commands (PostgreSQL)**

**Topic:** `CREATE`, `ALTER`, `RENAME`, `DROP`, and `TRUNCATE`
**Database:** `ddl_lab`
**Schema:** `company`

---

## 🎯 **Objectives**

By the end of this lab, you will be able to:

* Create a table using `CREATE TABLE`.
* Modify table structure using various `ALTER` commands.
* Rename columns and tables.
* Delete data using `TRUNCATE`.
* Delete database objects permanently using `DROP`.

---

## 🧰 **Setup**

1. Open **pgAdmin Query Tool** or **psql** terminal.
2. Create and connect to your lab database:

   ```sql
   CREATE DATABASE ddl_lab;
   \c ddl_lab
   ```
3. Create a new schema for this lab:

   ```sql
   CREATE SCHEMA company AUTHORIZATION postgres;
   ```

---

## 🧱 **Step 1 – CREATE TABLE**

```sql
CREATE TABLE company.employee (
    emp_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50),
    email VARCHAR(100) UNIQUE,
    salary NUMERIC(10,2),
    department VARCHAR(50) DEFAULT 'IT'
);
```

✅ **Verify Structure**

```sql
\d company.employee;
```

✅ **Insert Sample Data**

```sql
INSERT INTO company.employee (first_name, last_name, email, salary, department)
VALUES
('Varun', 'Gupta', 'varun@company.com', 85000, 'IT'),
('Neha', 'Sharma', 'neha@company.com', 90000, 'Finance');
```

✅ **View Data**

```sql
SELECT * FROM company.employee;
```

---

## 🔧 **Step 2 – ALTER TABLE (Core Practice)**

Let’s explore how to **change the structure** of the existing table.

---

### 🧩 2.1 **Add a New Column**

Add a `joining_date` column to store when the employee joined.

```sql
ALTER TABLE company.employee
ADD COLUMN joining_date DATE DEFAULT CURRENT_DATE;
```

✅ **Verify**

```sql
\d company.employee;
```

🧠 *Observation:* A new column appears with default value `current_date`.

---

### 🧩 2.2 **Add Multiple Columns at Once**

Add two columns: `phone` (text) and `city` (text).

```sql
ALTER TABLE company.employee
ADD COLUMN phone VARCHAR(15),
ADD COLUMN city VARCHAR(50);
```

✅ **Verify**

```sql
\d company.employee;
```

🧠 *Observation:* Both new columns appear in the table definition.

---

### 🧩 2.3 **Drop an Existing Column**

Remove the `department` column from the table.

```sql
ALTER TABLE company.employee
DROP COLUMN department;
```

✅ **Verify**

```sql
\d company.employee;
```

🧠 *Observation:* The `department` column is no longer listed.

---

### 🧩 2.4 **Rename a Column**

Rename `salary` to `base_salary`.

```sql
ALTER TABLE company.employee
RENAME COLUMN salary TO base_salary;
```

✅ **Verify**

```sql
\d company.employee;
```

🧠 *Observation:* The column name changes to `base_salary`.

---

### 🧩 2.5 **Change a Column’s Data Type**

Change the data type of `base_salary` from `NUMERIC(10,2)` to `INTEGER`.
When changing data types, PostgreSQL may need explicit conversion using `USING`.

```sql
ALTER TABLE company.employee
ALTER COLUMN base_salary TYPE INTEGER USING base_salary::INTEGER;
```

✅ **Verify**

```sql
SELECT column_name, data_type 
FROM information_schema.columns 
WHERE table_name = 'employee' AND table_schema = 'company';
```

🧠 *Observation:* `base_salary` is now stored as an integer.

---

### 🧩 2.6 **Rename the Table**

Rename the `employee` table to `staff`.

```sql
ALTER TABLE company.employee
RENAME TO staff;
```

✅ **Verify**

```sql
\d company.staff;
```

🧠 *Observation:* Table is now called `staff`.

---

### 🧩 2.7 **Rename the Schema (Optional)**

Rename the schema from `company` to `organization`.

```sql
ALTER SCHEMA company RENAME TO organization;
```

✅ **Verify**

```sql
\dn
```

🧠 *Observation:* The schema name is now `organization`.

---

## 🧹 **Step 3 – TRUNCATE TABLE**

Removes all rows instantly but **keeps the structure**.

```sql
TRUNCATE TABLE organization.staff;
```

✅ **Verify**

```sql
SELECT * FROM organization.staff;  -- should return 0 rows
```

🧠 *Observation:* Table is empty but columns remain.

---

## 💣 **Step 4 – DROP TABLE**

Deletes both data and table structure permanently.

```sql
DROP TABLE organization.staff;
```

✅ **Verify**

```sql
\d organization.*;  -- should show nothing
```

🧠 *Observation:* Table no longer exists.

---

## 🧾 **Summary Table**

| Operation                | Command Example                                                   | Description                     |
| ------------------------ | ----------------------------------------------------------------- | ------------------------------- |
| **Add column**           | `ALTER TABLE emp ADD COLUMN age INT;`                             | Adds a new column               |
| **Add multiple columns** | `ALTER TABLE emp ADD COLUMN city TEXT, ADD COLUMN state TEXT;`    | Adds several columns            |
| **Drop column**          | `ALTER TABLE emp DROP COLUMN city;`                               | Removes a column                |
| **Rename column**        | `ALTER TABLE emp RENAME COLUMN name TO full_name;`                | Renames a column                |
| **Change data type**     | `ALTER TABLE emp ALTER COLUMN salary TYPE INT USING salary::INT;` | Converts data type              |
| **Rename table**         | `ALTER TABLE emp RENAME TO staff;`                                | Changes table name              |
| **Rename schema**        | `ALTER SCHEMA old_name RENAME TO new_name;`                       | Renames a schema                |
| **Truncate**             | `TRUNCATE TABLE staff;`                                           | Clears data but keeps structure |
| **Drop**                 | `DROP TABLE staff;`                                               | Deletes structure and data      |

---

