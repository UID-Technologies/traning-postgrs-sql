
# 🧩 **Lab 04 – DML Commands (PostgreSQL)**

**Focus:** `INSERT`, `UPDATE`, `DELETE`
**Database:** `ddl_lab`
**Schema:** `organization`
**Table:** `staff`

---

## 🎯 **Objectives**

By the end of this lab, you will be able to:

* Add data into tables using `INSERT`.
* Modify existing data using `UPDATE`.
* Remove data using `DELETE`.
* Understand the impact of each DML operation.

---

## 🧰 **Setup**

If your table doesn’t exist, recreate it:

```sql
CREATE SCHEMA IF NOT EXISTS organization AUTHORIZATION postgres;

CREATE TABLE organization.staff (
    emp_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50),
    email VARCHAR(100),
    base_salary INTEGER,
    joining_date DATE,
    phone VARCHAR(15),
    city VARCHAR(50)
);
```

✅ **Verify Table**

```sql
\d organization.staff;
```

---

## 🧱 **Step 1 – INSERT Command**

The `INSERT` statement is used to add new records to a table.

### 🧩 1.1 Insert a Single Row

```sql
INSERT INTO organization.staff (first_name, last_name, email, base_salary, joining_date, phone, city)
VALUES ('Varun', 'Gupta', 'varun@company.com', 85000, '2022-05-15', '9876543210', 'Delhi');
```

✅ **Verify**

```sql
SELECT * FROM organization.staff;
```

---

### 🧩 1.2 Insert Multiple Rows at Once

```sql
INSERT INTO organization.staff (first_name, last_name, email, base_salary, joining_date, phone, city)
VALUES
('Neha', 'Sharma', 'neha@company.com', 90000, '2021-12-20', '9988776655', 'Mumbai'),
('Amit', 'Verma', 'amit@company.com', 75000, '2023-01-10', '9898989898', 'Delhi'),
('Priya', 'Patel', 'priya@company.com', 80000, '2020-07-05', '9900112233', 'Pune'),
('Ravi', 'Kumar', 'ravi@company.com', 95000, '2021-03-22', '9123456789', 'Bangalore');
```

✅ **Verify**

```sql
SELECT * FROM organization.staff;
```

---

### 🧩 1.3 Insert Partial Columns (Optional Columns Auto-NULL)

You can skip optional columns; PostgreSQL will fill them with `NULL` or default values.

```sql
INSERT INTO organization.staff (first_name, base_salary, city)
VALUES ('Anjali', 70000, 'Hyderabad');
```

✅ **Verify**

```sql
SELECT * FROM organization.staff;
```

🧠 *Observation:* Columns not specified (e.g., `email`, `joining_date`, `phone`) show `NULL`.

---

## ✏️ **Step 2 – UPDATE Command**

The `UPDATE` statement is used to modify existing rows in a table.

---

### 🧩 2.1 Update a Single Record

Increase salary for one specific employee.

```sql
UPDATE organization.staff
SET base_salary = 90000
WHERE first_name = 'Amit';
```

✅ **Verify**

```sql
SELECT first_name, base_salary FROM organization.staff WHERE first_name = 'Amit';
```

---

### 🧩 2.2 Update Multiple Columns Together

```sql
UPDATE organization.staff
SET city = 'New Delhi', phone = '9999999999'
WHERE first_name = 'Varun';
```

✅ **Verify**

```sql
SELECT first_name, city, phone FROM organization.staff WHERE first_name = 'Varun';
```

---

### 🧩 2.3 Update Multiple Rows (Using Condition)

Increase salary by ₹5,000 for all employees in Mumbai.

```sql
UPDATE organization.staff
SET base_salary = base_salary + 5000
WHERE city = 'Mumbai';
```

✅ **Verify**

```sql
SELECT first_name, city, base_salary FROM organization.staff WHERE city = 'Mumbai';
```

---

### 🧩 2.4 Update Using a Subquery (Optional Advanced)

Give everyone earning below the average salary a 10% raise.

```sql
UPDATE organization.staff
SET base_salary = base_salary * 1.10
WHERE base_salary < (SELECT AVG(base_salary) FROM organization.staff);
```

✅ **Verify**

```sql
SELECT first_name, base_salary FROM organization.staff;
```

---

## ❌ **Step 3 – DELETE Command**

The `DELETE` statement is used to remove records from a table.

---

### 🧩 3.1 Delete a Single Record

```sql
DELETE FROM organization.staff
WHERE first_name = 'Priya';
```

✅ **Verify**

```sql
SELECT * FROM organization.staff;
```

---

### 🧩 3.2 Delete Multiple Rows (Condition)

Remove all employees from “Delhi” or “New Delhi”.

```sql
DELETE FROM organization.staff
WHERE city IN ('Delhi', 'New Delhi');
```

✅ **Verify**

```sql
SELECT * FROM organization.staff;
```

---

### 🧩 3.3 Delete All Records (Table Reset)

Removes all rows but keeps structure.

```sql
DELETE FROM organization.staff;
```

✅ **Verify**

```sql
SELECT COUNT(*) FROM organization.staff;  -- should return 0
```

🧠 *Note:*
Use `TRUNCATE TABLE organization.staff;` for faster cleanup (non-logged).

---

## 🧾 **Summary Table**

| Command    | Purpose              | Example                                     |
| ---------- | -------------------- | ------------------------------------------- |
| **INSERT** | Add new rows         | `INSERT INTO staff VALUES (...)`            |
| **UPDATE** | Modify existing rows | `UPDATE staff SET salary=90000 WHERE id=1;` |
| **DELETE** | Remove rows          | `DELETE FROM staff WHERE city='Delhi';`     |

---

## ⚙️ **Practice Tasks**

1. Add 3 new employee records with different cities.
2. Increase everyone’s salary in Bangalore by 8%.
3. Change phone number format to include “+91” prefix for all.
4. Delete employees who joined before `2021-01-01`.

---

