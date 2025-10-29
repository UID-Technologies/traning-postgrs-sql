
# 🧩 **Lab 06 - Database Constraints (PostgreSQL)**

---

## 🎯 **Objective**

By the end of this lesson, you will:

* Understand what **constraints** are and why they are important.
* Learn the **types of constraints** used in relational databases.
* Practice creating and modifying tables with constraints in PostgreSQL.

---

## 🧠 **What is a Constraint?**

A **constraint** is a rule applied to a column or a table that **restricts the type of data** that can be stored.
Constraints ensure the **accuracy**, **validity**, and **integrity** of data inside the database.

👉 Think of them as *data quality rules* enforced by the database itself.

### Example:

If you set a constraint that “Age must be greater than 18”,
then PostgreSQL will **reject** any record where `Age < 18`.

---

## 🧱 **Types of Constraints**

| Constraint Type | Description                                                                            | Example                                      |
| --------------- | -------------------------------------------------------------------------------------- | -------------------------------------------- |
| **NOT NULL**    | Ensures that a column cannot have `NULL` value.                                        | `name VARCHAR(50) NOT NULL`                  |
| **UNIQUE**      | Ensures all values in a column are different (no duplicates).                          | `email VARCHAR(100) UNIQUE`                  |
| **PRIMARY KEY** | Uniquely identifies each record in a table. It combines `NOT NULL + UNIQUE`.           | `emp_id SERIAL PRIMARY KEY`                  |
| **FOREIGN KEY** | Enforces relationship between tables; value must exist in another table’s primary key. | `dept_id INT REFERENCES department(dept_id)` |
| **CHECK**       | Ensures values meet specific condition(s).                                             | `CHECK (salary > 0)`                         |
| **DEFAULT**     | Provides a default value when none is given.                                           | `status VARCHAR(10) DEFAULT 'Active'`        |

---

## 🧰 **Setup for Lab**

Database: `ddl_lab`
Schema: `organization`

```sql
CREATE SCHEMA IF NOT EXISTS organization AUTHORIZATION postgres;
```

---

# 🧪 **Lab Exercise – Practicing Constraints**

---

## 🧩 **Step 1 – Create a Table with Constraints**

```sql
CREATE TABLE organization.employee (
    emp_id SERIAL PRIMARY KEY,                  -- Primary Key (unique + not null)
    first_name VARCHAR(50) NOT NULL,            -- Cannot be NULL
    last_name VARCHAR(50),
    email VARCHAR(100) UNIQUE,                  -- No duplicate emails
    base_salary NUMERIC(10,2) CHECK (base_salary > 0),   -- Must be positive
    city VARCHAR(50) DEFAULT 'Not Assigned',    -- Default value
    joining_date DATE NOT NULL,                 -- Must be provided
    dept_id INT                                 -- Foreign Key (will add later)
);
```

✅ **Verify Structure**

```sql
\d organization.employee;
```

---

## 🧩 **Step 2 – Insert Valid Data**

```sql
INSERT INTO organization.employee
(first_name, last_name, email, base_salary, city, joining_date)
VALUES
('Varun', 'Gupta', 'varun@company.com', 85000, 'Delhi', '2022-05-15'),
('Neha', 'Sharma', 'neha@company.com', 90000, 'Mumbai', '2021-12-20');
```

✅ **Verify**

```sql
SELECT * FROM organization.employee;
```

---

## 🧩 **Step 3 – Test Constraint Violations**

Try inserting invalid data to observe constraint behavior.

### 🔹 Test 1: Violating NOT NULL

```sql
INSERT INTO organization.employee (first_name, email, base_salary, joining_date)
VALUES (NULL, 'test@company.com', 70000, '2022-09-15');
```

❌ Expected: Error → *“null value in column ‘first_name’ violates not-null constraint”*

---

### 🔹 Test 2: Violating UNIQUE

```sql
INSERT INTO organization.employee (first_name, email, base_salary, joining_date)
VALUES ('Amit', 'varun@company.com', 75000, '2022-03-12');
```

❌ Expected: Error → *“duplicate key value violates unique constraint”*

---

### 🔹 Test 3: Violating CHECK

```sql
INSERT INTO organization.employee (first_name, email, base_salary, joining_date)
VALUES ('Ravi', 'ravi@company.com', -50000, '2022-01-10');
```

❌ Expected: Error → *“new row violates check constraint ‘employee_base_salary_check’”*

---

### 🔹 Test 4: Using DEFAULT

```sql
INSERT INTO organization.employee (first_name, email, base_salary, joining_date)
VALUES ('Priya', 'priya@company.com', 88000, '2023-01-10');
```

✅ Expected: City column will show `'Not Assigned'` automatically.

---

## 🧩 **Step 4 – Add FOREIGN KEY Constraint**

1️⃣ Create the referenced table:

```sql
CREATE TABLE organization.department (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(50) UNIQUE NOT NULL
);
```

2️⃣ Insert sample departments:

```sql
INSERT INTO organization.department (dept_name)
VALUES ('IT'), ('Finance'), ('HR'), ('Sales');
```

3️⃣ Add foreign key to employee table:

```sql
ALTER TABLE organization.employee
ADD CONSTRAINT fk_dept
FOREIGN KEY (dept_id)
REFERENCES organization.department(dept_id);
```

✅ **Verify**

```sql
\d organization.employee;
```

---

### 🔹 Test 5: Violating FOREIGN KEY

```sql
INSERT INTO organization.employee (first_name, email, base_salary, joining_date, dept_id)
VALUES ('Asha', 'asha@company.com', 78000, '2021-12-12', 999);
```

❌ Expected: Error → *“insert or update on table ‘employee’ violates foreign key constraint”*

---

### 🔹 Test 6: Valid FOREIGN KEY

```sql
INSERT INTO organization.employee (first_name, email, base_salary, joining_date, dept_id)
VALUES ('Asha', 'asha@company.com', 78000, '2021-12-12', 1);
```

✅ Expected: Record inserted successfully.

---

## 🧩 **Step 5 – View Constraints**

List all constraints for the employee table:

```sql
SELECT conname AS constraint_name, contype AS type
FROM pg_constraint
WHERE conrelid = 'organization.employee'::regclass;
```

🧠 **Constraint Type Legend:**

* `p` = Primary Key
* `u` = Unique
* `f` = Foreign Key
* `c` = Check

---

## 🧩 **Step 6 – Drop or Modify Constraints (Optional)**

### Drop a constraint

```sql
ALTER TABLE organization.employee
DROP CONSTRAINT fk_dept;
```

### Add back the constraint

```sql
ALTER TABLE organization.employee
ADD CONSTRAINT fk_dept FOREIGN KEY (dept_id)
REFERENCES organization.department(dept_id);
```

---

## 🧾 **Summary**

| Constraint      | Ensures                          | Example                                  |
| --------------- | -------------------------------- | ---------------------------------------- |
| **NOT NULL**    | Value must exist                 | `first_name VARCHAR(50) NOT NULL`        |
| **UNIQUE**      | No duplicates                    | `email UNIQUE`                           |
| **PRIMARY KEY** | Unique + Not Null                | `emp_id SERIAL PRIMARY KEY`              |
| **FOREIGN KEY** | Valid reference to another table | `dept_id REFERENCES department(dept_id)` |
| **CHECK**       | Logical condition must be true   | `CHECK (salary > 0)`                     |
| **DEFAULT**     | Auto value if none provided      | `DEFAULT 'Active'`                       |

---

