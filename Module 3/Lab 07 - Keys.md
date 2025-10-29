
# 🧩 **Lab 07 - Keys & Relationships in RDBMS (PostgreSQL)**

---

## 🎯 **Objectives**

By the end of this module, learners will:

* Understand what **keys** are and why they are used in databases.
* Identify and create **Primary Key**, **Foreign Key**, **Composite Key**, and **Candidate Key**.
* Establish **relationships** between tables (One-to-One, One-to-Many, Many-to-Many).
* Practice defining and verifying these keys in PostgreSQL.

---

## 🧠 **What is a Key?**

A **Key** is a field (or combination of fields) used to **uniquely identify a record** in a table.
Keys ensure that data remains **consistent**, **non-duplicate**, and **logically connected** between tables.

---

## 🧱 **Types of Keys**

| Key Type             | Description                                                                                | Example                                         |
| -------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------- |
| **Primary Key (PK)** | Uniquely identifies each record in a table. Cannot be `NULL`.                              | `emp_id SERIAL PRIMARY KEY`                     |
| **Foreign Key (FK)** | References the Primary Key of another table to build relationships.                        | `dept_id REFERENCES department(dept_id)`        |
| **Composite Key**    | Combination of two or more columns that together uniquely identify a record.               | `(student_id, course_id)`                       |
| **Candidate Key**    | All possible columns that can serve as a Primary Key.                                      | `emp_id`, `email`                               |
| **Alternate Key**    | Candidate Key that was **not chosen** as the Primary Key.                                  | If `email` not used as PK, it becomes Alternate |
| **Super Key**        | Any combination of attributes that uniquely identifies a row (includes PK + extra fields). | `(emp_id, email)`                               |

---

## 🧩 **Relationships Between Tables**

| Relationship Type      | Description                                                      | Example                            |
| ---------------------- | ---------------------------------------------------------------- | ---------------------------------- |
| **One-to-One (1:1)**   | Each record in Table A relates to exactly one record in Table B. | Each employee has one profile.     |
| **One-to-Many (1:N)**  | One record in Table A relates to many records in Table B.        | One department → many employees.   |
| **Many-to-Many (M:N)** | Many records in A relate to many in B (requires junction table). | Students enrolled in many courses. |

---

# 🧪 **Lab Exercise – Keys & Relationships**

---

## 🧰 **Setup**

Database: `ddl_lab`
Schema: `organization`

### 1️⃣ Create the Department Table

```sql
CREATE TABLE organization.department (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(50) UNIQUE NOT NULL,
    location VARCHAR(50)
);
```

✅ **Verify**

```sql
\d organization.department;
```

### Insert Departments

```sql
INSERT INTO organization.department (dept_name, location)
VALUES ('IT', 'Delhi'),
       ('Finance', 'Mumbai'),
       ('HR', 'Bangalore');
```

---

### 2️⃣ Create the Employee Table (One-to-Many Relationship)

Each department has many employees, but each employee belongs to one department.

```sql
CREATE TABLE organization.employee (
    emp_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50),
    email VARCHAR(100) UNIQUE,
    base_salary NUMERIC(10,2),
    dept_id INT REFERENCES organization.department(dept_id)   -- Foreign Key
);
```

✅ **Verify**

```sql
\d organization.employee;
```

### Insert Employees

```sql
INSERT INTO organization.employee (first_name, last_name, email, base_salary, dept_id)
VALUES
('Varun', 'Gupta', 'varun@company.com', 85000, 1),
('Neha', 'Sharma', 'neha@company.com', 90000, 2),
('Ravi', 'Kumar', 'ravi@company.com', 95000, 1),
('Amit', 'Verma', 'amit@company.com', 75000, 3);
```

✅ **Verify Data**

```sql
SELECT e.emp_id, e.first_name, e.email, d.dept_name
FROM organization.employee e
JOIN organization.department d ON e.dept_id = d.dept_id;
```

---

## 🔑 **Step 1 – Primary Key Practice**

Try creating a table without a primary key and then adding one.

### Without PK

```sql
CREATE TABLE organization.temp_staff (
    staff_id INT,
    name VARCHAR(50)
);
```

### Add Primary Key Later

```sql
ALTER TABLE organization.temp_staff
ADD CONSTRAINT pk_temp_staff PRIMARY KEY (staff_id);
```

✅ **Verify**

```sql
\d organization.temp_staff;
```

---

## 🔗 **Step 2 – Foreign Key Practice**

Add a foreign key to link tables together.

### Create a new table referencing Employee

```sql
CREATE TABLE organization.salary (
    salary_id SERIAL PRIMARY KEY,
    emp_id INT NOT NULL,
    monthly_salary NUMERIC(10,2) NOT NULL,
    FOREIGN KEY (emp_id) REFERENCES organization.employee(emp_id)
);
```

### Insert Records

```sql
INSERT INTO organization.salary (emp_id, monthly_salary)
VALUES (1, 85000), (2, 90000), (3, 95000);
```

✅ **Verify**

```sql
SELECT s.salary_id, e.first_name, s.monthly_salary
FROM organization.salary s
JOIN organization.employee e ON s.emp_id = e.emp_id;
```

### Test Foreign Key Violation

```sql
INSERT INTO organization.salary (emp_id, monthly_salary)
VALUES (99, 80000);  -- Invalid employee
```

❌ Expected: *violates foreign key constraint*

---

## 🧮 **Step 3 – Composite Key Practice**

When a single column isn’t unique enough, combine two or more columns.

### Example: Employee Attendance Table

Each employee can have many attendance records, but each date should be unique per employee.

```sql
CREATE TABLE organization.attendance (
    emp_id INT REFERENCES organization.employee(emp_id),
    attendance_date DATE,
    status VARCHAR(10) DEFAULT 'Present',
    PRIMARY KEY (emp_id, attendance_date)   -- Composite Key
);
```

✅ **Insert Data**

```sql
INSERT INTO organization.attendance (emp_id, attendance_date, status)
VALUES 
(1, '2025-10-01', 'Present'),
(1, '2025-10-02', 'Absent'),
(2, '2025-10-01', 'Present');
```

✅ **Test Duplicate Record**

```sql
INSERT INTO organization.attendance (emp_id, attendance_date, status)
VALUES (1, '2025-10-01', 'Present');  -- Duplicate PK
```

❌ Expected: *violates primary key constraint*

---

## 🔁 **Step 4 – Many-to-Many Relationship**

Example: Employees working on multiple projects.

### Create Projects Table

```sql
CREATE TABLE organization.project (
    project_id SERIAL PRIMARY KEY,
    project_name VARCHAR(50) UNIQUE
);
```

### Create Junction Table

```sql
CREATE TABLE organization.employee_project (
    emp_id INT REFERENCES organization.employee(emp_id),
    project_id INT REFERENCES organization.project(project_id),
    role VARCHAR(50),
    PRIMARY KEY (emp_id, project_id)
);
```

### Insert Data

```sql
INSERT INTO organization.project (project_name)
VALUES ('LMS Platform'), ('Data Warehouse'), ('HR Automation');

INSERT INTO organization.employee_project (emp_id, project_id, role)
VALUES 
(1, 1, 'Developer'),
(1, 2, 'Lead'),
(2, 1, 'Tester'),
(3, 3, 'Architect');
```

✅ **Verify Many-to-Many Join**

```sql
SELECT e.first_name, p.project_name, ep.role
FROM organization.employee_project ep
JOIN organization.employee e ON ep.emp_id = e.emp_id
JOIN organization.project p ON ep.project_id = p.project_id;
```

---

## 🧾 **Summary**

| Key Type          | Ensures                         | Example                                  |
| ----------------- | ------------------------------- | ---------------------------------------- |
| **Primary Key**   | Each record is unique           | `emp_id SERIAL PRIMARY KEY`              |
| **Foreign Key**   | Maintains referential integrity | `dept_id REFERENCES department(dept_id)` |
| **Composite Key** | Unique combination of columns   | `PRIMARY KEY(emp_id, date)`              |
| **Candidate Key** | All possible unique identifiers | `emp_id`, `email`                        |
| **Alternate Key** | Candidate key not used as PK    | `email` if PK is `emp_id`                |

---

✅ Deliverables

Each learner should:

1. Create all 4 tables (department, employee, salary, project).

2. Demonstrate working Primary, Foreign, and Composite Keys.

3. Capture:

 - Table structure (\d organization.table_name)

 - Query outputs for valid & invalid inserts.

