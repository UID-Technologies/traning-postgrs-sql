
# 🧩 **Lab 09 -  Aggregate Functions, GROUP BY, and HAVING (PostgreSQL)**

---

## 🎯 **Objectives**

By the end of this module, learners will:

* Understand and use SQL **aggregate functions** (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`).
* Group data using the `GROUP BY` clause.
* Filter grouped results with the `HAVING` clause.
* Combine aggregate functions with joins for practical analytics.

---

## 🧠 **What Are Aggregate Functions?**

Aggregate functions perform **calculations on multiple rows** and return a **single value** per group or dataset.

For example:

> “What is the average salary of employees per department?”

Aggregate functions ignore `NULL` values unless specified otherwise.

---

## 📊 **Common Aggregate Functions**

| Function    | Description            | Example            |
| ----------- | ---------------------- | ------------------ |
| **COUNT()** | Returns number of rows | `COUNT(*)`         |
| **SUM()**   | Adds up numeric values | `SUM(base_salary)` |
| **AVG()**   | Returns average value  | `AVG(base_salary)` |
| **MIN()**   | Finds minimum value    | `MIN(base_salary)` |
| **MAX()**   | Finds maximum value    | `MAX(base_salary)` |

---

## 🧰 **Setup**

Use your existing tables from previous labs:

* `organization.department`
* `organization.employee`
* `organization.salary`

If not available, recreate minimal versions:

```sql
CREATE SCHEMA IF NOT EXISTS organization AUTHORIZATION postgres;

CREATE TABLE organization.department (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(50) UNIQUE NOT NULL,
    location VARCHAR(50)
);

CREATE TABLE organization.employee (
    emp_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    dept_id INT REFERENCES organization.department(dept_id)
);

CREATE TABLE organization.salary (
    salary_id SERIAL PRIMARY KEY,
    emp_id INT REFERENCES organization.employee(emp_id),
    monthly_salary NUMERIC(10,2)
);
```

Insert sample data:

```sql
INSERT INTO organization.department (dept_name, location)
VALUES ('IT','Delhi'), ('Finance','Mumbai'), ('HR','Bangalore');

INSERT INTO organization.employee (first_name, last_name, dept_id)
VALUES
('Varun', 'Gupta', 1),
('Neha', 'Sharma', 2),
('Amit', 'Verma', 1),
('Priya', 'Patel', 3),
('Ravi', 'Kumar', 1);

INSERT INTO organization.salary (emp_id, monthly_salary)
VALUES
(1, 85000), (2, 90000), (3, 75000), (4, 80000), (5, 95000);
```

✅ **Verify Data**

```sql
SELECT e.first_name, d.dept_name, s.monthly_salary
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id
JOIN organization.salary s ON e.emp_id = s.emp_id;
```

---

# 🧪 **Lab Exercises – Aggregate Functions**

---

## 🔹 **Step 1 – COUNT()**

### 1.1 Count total employees

```sql
SELECT COUNT(*) AS total_employees
FROM organization.employee;
```

### 1.2 Count employees per department

```sql
SELECT d.dept_name, COUNT(e.emp_id) AS total_employees
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id
GROUP BY d.dept_name;
```

---

## 🔹 **Step 2 – SUM()**

### 2.1 Total salary expenditure

```sql
SELECT SUM(monthly_salary) AS total_salary_cost
FROM organization.salary;
```

### 2.2 Salary sum by department

```sql
SELECT d.dept_name, SUM(s.monthly_salary) AS total_salary
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id
JOIN organization.salary s ON e.emp_id = s.emp_id
GROUP BY d.dept_name;
```

---

## 🔹 **Step 3 – AVG()**

### 3.1 Average salary across organization

```sql
SELECT AVG(monthly_salary) AS average_salary
FROM organization.salary;
```

### 3.2 Average salary by department

```sql
SELECT d.dept_name, ROUND(AVG(s.monthly_salary), 2) AS avg_salary
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id
JOIN organization.salary s ON e.emp_id = s.emp_id
GROUP BY d.dept_name;
```

---

## 🔹 **Step 4 – MIN() and MAX()**

### 4.1 Find the highest and lowest salaries

```sql
SELECT MAX(monthly_salary) AS highest_salary,
       MIN(monthly_salary) AS lowest_salary
FROM organization.salary;
```

### 4.2 Highest salary per department

```sql
SELECT d.dept_name, MAX(s.monthly_salary) AS max_salary
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id
JOIN organization.salary s ON e.emp_id = s.emp_id
GROUP BY d.dept_name;
```

---

## 🔹 **Step 5 – Using GROUP BY**

### Group employees by department and count them

```sql
SELECT d.dept_name, COUNT(e.emp_id) AS emp_count
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id
GROUP BY d.dept_name;
```

### Group by multiple columns

```sql
SELECT d.dept_name, e.dept_id, COUNT(*) AS total
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id
GROUP BY d.dept_name, e.dept_id;
```

---

## 🔹 **Step 6 – Using HAVING Clause**

The **HAVING** clause filters groups after aggregation (similar to WHERE, but used with `GROUP BY`).

### 6.1 Departments with more than 1 employee

```sql
SELECT d.dept_name, COUNT(e.emp_id) AS total_employees
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id
GROUP BY d.dept_name
HAVING COUNT(e.emp_id) > 1;
```

### 6.2 Departments where total salary > 200000

```sql
SELECT d.dept_name, SUM(s.monthly_salary) AS total_salary
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id
JOIN organization.salary s ON e.emp_id = s.emp_id
GROUP BY d.dept_name
HAVING SUM(s.monthly_salary) > 200000;
```

---

## 🔹 **Step 7 – Combine Aggregates with Filtering**

### 7.1 Average salary of employees joined to “IT” department

```sql
SELECT d.dept_name, ROUND(AVG(s.monthly_salary), 2) AS avg_salary
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id
JOIN organization.salary s ON e.emp_id = s.emp_id
WHERE d.dept_name = 'IT'
GROUP BY d.dept_name;
```

---

## 🧾 **Summary Table**

| Function     | Description            | Example                       |
| ------------ | ---------------------- | ----------------------------- |
| **COUNT()**  | Counts rows            | `COUNT(*)`                    |
| **SUM()**    | Total sum              | `SUM(salary)`                 |
| **AVG()**    | Average value          | `AVG(salary)`                 |
| **MIN()**    | Smallest value         | `MIN(salary)`                 |
| **MAX()**    | Largest value          | `MAX(salary)`                 |
| **GROUP BY** | Groups data by column  | `GROUP BY dept_name`          |
| **HAVING**   | Filters after grouping | `HAVING SUM(salary) > 200000` |

---

## ✅ **Deliverables**

Each learner should:

1. Execute all examples for `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.
2. Run `GROUP BY` and `HAVING` queries with valid output.
3. Submit:

   * Table structures (`\d organization.*`)
   * Query results screenshots
   * A short summary of difference between `WHERE` and `HAVING`.

---

## ⚙️ **Practice Challenges**

1. Find the **department with the highest total salary**.
2. Display **total employees and average salary per department**.
3. Show **departments having total salary less than 200000**.
4. Find the **minimum, maximum, and average salary in IT**.
5. Find **number of employees with salary above organization average**.

