
# 🧩 **Lab 08 - SQL JOINs (PostgreSQL Hands-on Lab)**

---

## 🎯 **Objectives**

By the end of this module, learners will:

* Understand what **JOINs** are and why they are used.
* Learn the difference between **INNER**, **LEFT**, **RIGHT**, **FULL**, **SELF**, and **CROSS JOIN**.
* Practice joining multiple tables using **real relationships**.
* Visualize data relationships using SQL queries.

---

## 🧠 **What is a JOIN?**

A **JOIN** is used to combine data from **two or more tables** based on a related column between them.

👉 Example:
Get employee names and their department names using a shared column `dept_id`.

---

## 🧱 **Common Types of JOINs**

| JOIN Type      | Description                                                          | Returns                        |
| -------------- | -------------------------------------------------------------------- | ------------------------------ |
| **INNER JOIN** | Returns rows that have matching values in both tables.               | Only matching records          |
| **LEFT JOIN**  | Returns all rows from the left table, even if no match in the right. | All left + matching right      |
| **RIGHT JOIN** | Returns all rows from the right table, even if no match in the left. | All right + matching left      |
| **FULL JOIN**  | Returns all rows when there is a match in one or both tables.        | All rows (matched + unmatched) |
| **SELF JOIN**  | Joins a table to itself.                                             | Used for hierarchical data     |
| **CROSS JOIN** | Returns Cartesian product of both tables.                            | All combinations of rows       |

---

## 🧰 **Setup**

You already have:

* `organization.department`
* `organization.employee`
* `organization.salary`
* `organization.project`
* `organization.employee_project`

If not, quickly recreate them from the **previous module**.

---

# 🧪 **Lab Exercises – SQL JOINs**

---

## 🔹 **Step 1 – INNER JOIN**

Retrieve employees along with their department names.

```sql
SELECT e.emp_id, e.first_name, e.email, d.dept_name
FROM organization.employee e
INNER JOIN organization.department d
ON e.dept_id = d.dept_id;
```

✅ **Observation:**
Only employees with a valid department are shown.

---

### Example 2: Join 3 tables (employee + department + salary)

```sql
SELECT e.first_name, d.dept_name, s.monthly_salary
FROM organization.employee e
INNER JOIN organization.department d ON e.dept_id = d.dept_id
INNER JOIN organization.salary s ON e.emp_id = s.emp_id;
```

✅ **Observation:**
Shows employees with both department and salary records.

---

## 🔹 **Step 2 – LEFT JOIN**

Return all employees even if they don’t have a department assigned.

```sql
SELECT e.first_name, e.dept_id, d.dept_name
FROM organization.employee e
LEFT JOIN organization.department d
ON e.dept_id = d.dept_id;
```

✅ **Observation:**
If a department is missing, `dept_name` will show `NULL`.

---

### Example 2: Employee and Salary (including employees without salary)

```sql
SELECT e.first_name, s.monthly_salary
FROM organization.employee e
LEFT JOIN organization.salary s
ON e.emp_id = s.emp_id;
```

✅ **Observation:**
Shows all employees; salary column will be `NULL` if not found.

---

## 🔹 **Step 3 – RIGHT JOIN**

Return all departments, even if no employees exist in them.

```sql
SELECT e.first_name, d.dept_name
FROM organization.employee e
RIGHT JOIN organization.department d
ON e.dept_id = d.dept_id;
```

✅ **Observation:**
All departments appear; employees may be `NULL` for empty departments.

---

## 🔹 **Step 4 – FULL JOIN**

Combine results of `LEFT` and `RIGHT` joins.
Shows all employees and departments — matched or not.

```sql
SELECT e.first_name, d.dept_name
FROM organization.employee e
FULL JOIN organization.department d
ON e.dept_id = d.dept_id;
```

✅ **Observation:**
All rows from both tables; unmatched ones have `NULL` values.

---

## 🔹 **Step 5 – SELF JOIN**

Use case: find employees in the same department (comparing a table with itself).

```sql
SELECT e1.first_name AS employee_name,
       e2.first_name AS colleague_name,
       e1.dept_id
FROM organization.employee e1
JOIN organization.employee e2
ON e1.dept_id = e2.dept_id
WHERE e1.emp_id <> e2.emp_id
ORDER BY e1.dept_id;
```

✅ **Observation:**
Pairs of employees who work in the same department.

---

## 🔹 **Step 6 – CROSS JOIN**

Returns all combinations (Cartesian product).
⚠️ Use carefully (can return a large number of rows).

```sql
SELECT e.first_name, p.project_name
FROM organization.employee e
CROSS JOIN organization.project p;
```

✅ **Observation:**
Each employee appears once for every project.

---

## 🔹 **Step 7 – Many-to-Many Join**

Retrieve employees and their assigned projects.

```sql
SELECT e.first_name, p.project_name, ep.role
FROM organization.employee_project ep
JOIN organization.employee e ON ep.emp_id = e.emp_id
JOIN organization.project p ON ep.project_id = p.project_id;
```

✅ **Observation:**
Displays which employee works on which project and their role.

---

## 🔹 **Step 8 – Filtering Joined Results**

Get employees from **‘IT’ department** earning more than 80,000.

```sql
SELECT e.first_name, d.dept_name, s.monthly_salary
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id
JOIN organization.salary s ON e.emp_id = s.emp_id
WHERE d.dept_name = 'IT' AND s.monthly_salary > 80000;
```

✅ **Observation:**
Shows filtered data after join conditions.

---

## 🧾 **Summary Table**

| JOIN Type      | Returns                   | Example                          |
| -------------- | ------------------------- | -------------------------------- |
| **INNER JOIN** | Only matching records     | `employee ↔ department`          |
| **LEFT JOIN**  | All from left + matches   | `employee LEFT JOIN department`  |
| **RIGHT JOIN** | All from right + matches  | `employee RIGHT JOIN department` |
| **FULL JOIN**  | All from both sides       | `employee FULL JOIN department`  |
| **SELF JOIN**  | Table joined with itself  | Compare employees in same dept   |
| **CROSS JOIN** | All possible combinations | `employee CROSS JOIN project`    |

---

## ✅ **Deliverables**

Each learner should:

1. Run all 6 JOIN queries.
2. Take screenshots of:

   * Output of INNER, LEFT, RIGHT, and FULL joins.
   * SELF JOIN results showing colleague pairs.
   * CROSS JOIN output with limited rows (`LIMIT 10`).
3. Submit one `.sql` file or screenshots showing all JOIN outputs.

---

## 🧩 **Practice Challenges**

1. List employees who do **not** have a department assigned.
2. Find total salary per department using JOIN + `GROUP BY`.
3. Show project names and count of employees working on each.
4. Find employees working in **“Finance”** but not assigned to any project.

