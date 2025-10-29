# 🧩 **Lab 05 – DQL Commands (PostgreSQL)**

**Focus:** `SELECT`, `WHERE`, `ORDER BY`, `DISTINCT`, `BETWEEN`, `LIKE`, `IN`, `LIMIT`
**Database:** `ddl_lab`
**Schema:** `organization`
**Table:** `staff`

---

## 🎯 **Objectives**

By the end of this lab, you will:

* Retrieve and display data using `SELECT`.
* Filter results using conditions (`WHERE`).
* Sort, limit, and format query results.
* Use operators like `BETWEEN`, `LIKE`, `IN`, and `DISTINCT`.

---

## 🧰 **Setup**

If table doesn’t exist, recreate it and insert some data:

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

### 🧩 Insert Sample Data

```sql
INSERT INTO organization.staff (first_name, last_name, email, base_salary, joining_date, phone, city)
VALUES
('Varun', 'Gupta', 'varun@company.com', 85000, '2022-05-15', '9876543210', 'Delhi'),
('Neha', 'Sharma', 'neha@company.com', 90000, '2021-12-20', '9988776655', 'Mumbai'),
('Amit', 'Verma', 'amit@company.com', 75000, '2023-01-10', '9898989898', 'Delhi'),
('Priya', 'Patel', 'priya@company.com', 80000, '2020-07-05', '9900112233', 'Pune'),
('Ravi', 'Kumar', 'ravi@company.com', 95000, '2021-03-22', '9123456789', 'Bangalore'),
('Anjali', 'Singh', 'anjali@company.com', 70000, '2022-09-14', '9345678901', 'Hyderabad'),
('Raj', 'Mehta', 'raj@company.com', 99000, '2020-11-11', '9321456789', 'Mumbai');
```

✅ **Verify Data**

```sql
SELECT * FROM organization.staff;
```

---

## 🔍 **Step 1 – Basic SELECT**

### 1.1 Select all columns

```sql
SELECT * FROM organization.staff;
```

### 1.2 Select specific columns

```sql
SELECT first_name, city, base_salary FROM organization.staff;
```

### 1.3 Add column aliases

```sql
SELECT first_name AS "Employee Name",
       city AS "Location",
       base_salary AS "Monthly Salary"
FROM organization.staff;
```

---

## 🎯 **Step 2 – Filtering with WHERE Clause**

### 2.1 Simple equality condition

```sql
SELECT * FROM organization.staff
WHERE city = 'Mumbai';
```

### 2.2 Comparison operators

```sql
SELECT first_name, base_salary FROM organization.staff
WHERE base_salary > 85000;
```

### 2.3 Multiple conditions (AND / OR)

```sql
SELECT first_name, city, base_salary
FROM organization.staff
WHERE city = 'Delhi' AND base_salary > 80000;
```

```sql
SELECT first_name, city, base_salary
FROM organization.staff
WHERE city = 'Delhi' OR city = 'Pune';
```

### 2.4 NOT operator

```sql
SELECT first_name, city
FROM organization.staff
WHERE NOT city = 'Mumbai';
```

---

## 🔢 **Step 3 – Range Filtering with BETWEEN**

### 3.1 Numeric range

```sql
SELECT first_name, base_salary
FROM organization.staff
WHERE base_salary BETWEEN 80000 AND 95000;
```

### 3.2 Date range

```sql
SELECT first_name, joining_date
FROM organization.staff
WHERE joining_date BETWEEN '2021-01-01' AND '2022-12-31';
```

---

## 🔡 **Step 4 – Pattern Matching with LIKE**

### 4.1 Starts with specific letter

```sql
SELECT first_name, email FROM organization.staff
WHERE first_name LIKE 'A%';  -- starts with A
```

### 4.2 Ends with specific letter

```sql
SELECT first_name, email FROM organization.staff
WHERE first_name LIKE '%a';  -- ends with 'a'
```

### 4.3 Contains pattern

```sql
SELECT first_name, city FROM organization.staff
WHERE city LIKE '%a%';  -- contains letter 'a'
```

---

## 📋 **Step 5 – Filter with IN / NOT IN**

### 5.1 Using IN (multiple matches)

```sql
SELECT first_name, city, base_salary
FROM organization.staff
WHERE city IN ('Delhi', 'Mumbai', 'Pune');
```

### 5.2 Using NOT IN (exclude some)

```sql
SELECT first_name, city, base_salary
FROM organization.staff
WHERE city NOT IN ('Delhi', 'Pune');
```

---

## 🔁 **Step 6 – Removing Duplicates with DISTINCT**

### 6.1 Show all unique cities

```sql
SELECT DISTINCT city FROM organization.staff;
```

### 6.2 Count how many distinct cities employees are from

```sql
SELECT COUNT(DISTINCT city) AS unique_city_count FROM organization.staff;
```

---

## 📊 **Step 7 – Sorting with ORDER BY**

### 7.1 Sort by salary ascending

```sql
SELECT first_name, base_salary FROM organization.staff
ORDER BY base_salary ASC;
```

### 7.2 Sort by salary descending

```sql
SELECT first_name, base_salary FROM organization.staff
ORDER BY base_salary DESC;
```

### 7.3 Sort by multiple columns

```sql
SELECT first_name, city, base_salary FROM organization.staff
ORDER BY city ASC, base_salary DESC;
```

---

## 🔢 **Step 8 – Limiting Results**

### 8.1 Fetch top 3 employees (highest salary)

```sql
SELECT first_name, base_salary
FROM organization.staff
ORDER BY base_salary DESC
LIMIT 3;
```

### 8.2 Skip first 2 and show next 3

```sql
SELECT first_name, base_salary
FROM organization.staff
ORDER BY base_salary DESC
OFFSET 2 LIMIT 3;
```

---

## 🧠 **Step 9 – Practice Mix Queries**

Try these on your own:

1. Find all employees who joined **after 2021** and earn **more than 80,000**.

   ```sql
   SELECT first_name, city, base_salary, joining_date
   FROM organization.staff
   WHERE joining_date > '2021-01-01' AND base_salary > 80000;
   ```

2. Show names of employees **not in Mumbai or Pune**, sorted by salary descending.

   ```sql
   SELECT first_name, city, base_salary
   FROM organization.staff
   WHERE city NOT IN ('Mumbai', 'Pune')
   ORDER BY base_salary DESC;
   ```

3. Retrieve only those whose names start with `R` or end with `a`.

   ```sql
   SELECT first_name, city
   FROM organization.staff
   WHERE first_name LIKE 'R%' OR first_name LIKE '%a';
   ```

4. Show top 2 highest-paid employees from each city (try using subquery or manual filter).

---

## 🧾 **Summary Table**

| Clause             | Purpose                 | Example                                 |
| ------------------ | ----------------------- | --------------------------------------- |
| **SELECT**         | Retrieve data           | `SELECT * FROM staff;`                  |
| **WHERE**          | Filter records          | `WHERE city='Delhi';`                   |
| **BETWEEN**        | Filter by range         | `WHERE salary BETWEEN 80000 AND 90000;` |
| **LIKE**           | Match patterns          | `WHERE name LIKE 'A%';`                 |
| **IN / NOT IN**    | Match from a list       | `WHERE city IN ('Delhi','Mumbai');`     |
| **DISTINCT**       | Remove duplicates       | `SELECT DISTINCT city;`                 |
| **ORDER BY**       | Sort results            | `ORDER BY salary DESC;`                 |
| **LIMIT / OFFSET** | Restrict number of rows | `LIMIT 5 OFFSET 2;`                     |

