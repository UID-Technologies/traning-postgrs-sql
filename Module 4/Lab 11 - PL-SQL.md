
# 🧩 **Lab 11 – PL/pgSQL Programming (PostgreSQL Hands-on Lab)**

---

## 🎯 **Objectives**

By the end of this lab, learners will:

* Understand what **PL/pgSQL** is and how it extends standard SQL.
* Declare and use **variables and data types**.
* Implement **conditional logic** (`IF/ELSIF/ELSE`) and **loops** (`FOR`, `WHILE`).
* Create and execute **functions** and **procedures**.
* Handle **exceptions** gracefully.
* Work with **cursors** for pagination and data iteration.
* Apply all concepts through practical exercises.

---

## 🧠 **What is PL/pgSQL?**

**PL/pgSQL (PostgreSQL Procedural Language)** is PostgreSQL’s built-in language that allows procedural logic inside the database.
You can use variables, loops, conditionals, and error handling — making it ideal for stored functions and business rules.

---

## 🧰 **Setup**

1. Connect to your PostgreSQL container:

   ```bash
  docker exec -it postgres-container bash
   psql -U postgres
   ```

2. Create a fresh training database:

   ```sql
   CREATE DATABASE pllab;
   \c pllab;
   ```

3. Create a base schema and tables:

   ```sql
   CREATE SCHEMA hr;

   CREATE TABLE hr.employees (
     emp_id SERIAL PRIMARY KEY,
     full_name TEXT NOT NULL,
     dept TEXT,
     salary NUMERIC(10,2),
     rating INT DEFAULT 3
   );

   INSERT INTO hr.employees (full_name, dept, salary, rating) VALUES
     ('Asha Singh','IT',1200000,5),
     ('Rohit Verma','IT',950000,4),
     ('Karan Shah','Finance',650000,3),
     ('Pooja Das','Finance',450000,2),
     ('Neha Gupta','HR',500000,3);
   ```

---

# 🧪 **Lab Exercises – PL/pgSQL Concepts**

---

## 🔹 **Step 1 – Variables and Data Types**

### Goal

Learn to declare variables and assign values.

```sql
DO $$
DECLARE
  v_count INT;
  v_avg   NUMERIC(10,2);
BEGIN
  SELECT COUNT(*), AVG(salary) INTO v_count, v_avg FROM hr.employees;
  RAISE NOTICE 'Total employees = %, Average Salary = %', v_count, v_avg;
END$$;
```

✅ **Observation:**
Shows computed values using PL/pgSQL variables.

---

## 🔹 **Step 2 – Conditional Statements (IF/ELSIF/ELSE)**

```sql
DO $$
DECLARE
  v_rating INT := 4;
BEGIN
  IF v_rating >= 5 THEN
    RAISE NOTICE 'Excellent Performer';
  ELSIF v_rating >= 3 THEN
    RAISE NOTICE 'Meets Expectations';
  ELSE
    RAISE NOTICE 'Needs Improvement';
  END IF;
END$$;
```

✅ **Observation:**
Conditional logic executes based on variable value.

---

## 🔹 **Step 3 – Loops (FOR, WHILE)**

```sql
DO $$
DECLARE
  rec RECORD;
  counter INT := 1;
BEGIN
  -- WHILE loop
  WHILE counter <= 3 LOOP
    RAISE NOTICE 'WHILE Loop Counter = %', counter;
    counter := counter + 1;
  END LOOP;

  -- FOR loop over query
  FOR rec IN SELECT emp_id, full_name FROM hr.employees LOOP
    RAISE NOTICE 'Employee %: %', rec.emp_id, rec.full_name;
  END LOOP;
END$$;
```

✅ **Observation:**
Demonstrates iteration through counters and query results.

---

## 🔹 **Step 4 – Function with IN/OUT Parameters**

### Example – Bonus Calculation Function

```sql
CREATE OR REPLACE FUNCTION hr.calc_bonus(
  p_salary NUMERIC,
  p_rating INT,
  OUT bonus_amount NUMERIC
) LANGUAGE plpgsql AS $$
BEGIN
  IF p_rating >= 5 THEN
    bonus_amount := p_salary * 0.20;
  ELSIF p_rating = 4 THEN
    bonus_amount := p_salary * 0.12;
  ELSIF p_rating = 3 THEN
    bonus_amount := p_salary * 0.06;
  ELSE
    bonus_amount := 0;
  END IF;
END$$;

-- Test
SELECT full_name, salary, rating,
       hr.calc_bonus(salary, rating) AS bonus
FROM hr.employees;
```

✅ **Observation:**
Each employee’s bonus is computed based on rating.

---

## 🔹 **Step 5 – Exception Handling**

```sql
DO $$
DECLARE
  a INT := 10;
  b INT := 0;
  c NUMERIC;
BEGIN
  BEGIN
    c := a / b;
  EXCEPTION WHEN division_by_zero THEN
    RAISE NOTICE 'Divide-by-zero handled!';
    c := NULL;
  END;
  RAISE NOTICE 'Result = %', c;
END$$;
```

✅ **Observation:**
No crash — error handled gracefully.

---

## 🔹 **Step 6 – Working with Cursors**

```sql
DO $$
DECLARE
  cur REFCURSOR;
  rec RECORD;
BEGIN
  OPEN cur FOR SELECT emp_id, full_name, salary FROM hr.employees ORDER BY emp_id;
  LOOP
    FETCH cur INTO rec;
    EXIT WHEN NOT FOUND;
    RAISE NOTICE 'EmpID:% Name:% Salary:%', rec.emp_id, rec.full_name, rec.salary;
  END LOOP;
  CLOSE cur;
END$$;
```

✅ **Observation:**
Fetches rows one by one using cursor.

---

## 🔹 **Step 7 – Stored Procedure Pattern (Batch Payroll Update)**

```sql
CREATE OR REPLACE PROCEDURE hr.update_payroll(p_dept TEXT, p_raise NUMERIC)
LANGUAGE plpgsql AS $$
DECLARE
  v_count INT;
BEGIN
  UPDATE hr.employees
     SET salary = salary * (1 + p_raise)
   WHERE dept = p_dept;
  GET DIAGNOSTICS v_count = ROW_COUNT;
  RAISE NOTICE '% employees in % got a % %% raise', v_count, p_dept, p_raise * 100;
END$$;

-- Execute
CALL hr.update_payroll('IT', 0.05);
```

✅ **Observation:**
Procedure updates and reports affected rows.

---

## 🔹 **Step 8 – Cursor-Based Pagination**

```sql
CREATE OR REPLACE FUNCTION hr.paginate_employees(
  p_last_id INT DEFAULT 0,
  p_limit INT DEFAULT 2
) RETURNS TABLE(emp_id INT, full_name TEXT, salary NUMERIC) AS $$
BEGIN
  RETURN QUERY
  SELECT emp_id, full_name, salary
  FROM hr.employees
  WHERE emp_id > p_last_id
  ORDER BY emp_id
  LIMIT p_limit;
END$$ LANGUAGE plpgsql;

-- Test
SELECT * FROM hr.paginate_employees(0, 2);  -- page 1  
SELECT * FROM hr.paginate_employees(2, 2);  -- page 2
```

✅ **Observation:**
Implements key-set pagination using function.

---

## 🔹 **Step 9 – Handle Divide-By-Zero Gracefully**

```sql
CREATE OR REPLACE FUNCTION hr.safe_ratio(
  p_num NUMERIC,
  p_den NUMERIC
) RETURNS NUMERIC AS $$
DECLARE
  v_result NUMERIC;
BEGIN
  BEGIN
    v_result := p_num / p_den;
  EXCEPTION WHEN division_by_zero THEN
    v_result := NULL;
  END;
  RETURN v_result;
END$$ LANGUAGE plpgsql;

-- Test
SELECT hr.safe_ratio(10, 2); -- 5  
SELECT hr.safe_ratio(10, 0); -- NULL
```

✅ **Observation:**
Function handles arithmetic errors safely.

---

## 🧾 **Summary Table**

| Concept     | Example Topic           | Command Used                |
| ----------- | ----------------------- | --------------------------- |
| Variables   | Declaring and Assigning | `DECLARE`, `INTO`           |
| Conditional | IF/ELSIF/ELSE           | `IF` logic                  |
| Loops       | Iteration over rows     | `FOR`, `WHILE`              |
| Function    | Bonus Calculator        | `CREATE FUNCTION`           |
| Procedure   | Payroll Update          | `CREATE PROCEDURE` + `CALL` |
| Exception   | Error Handling          | `BEGIN … EXCEPTION WHEN`    |
| Cursor      | Data Pagination         | `OPEN`, `FETCH`, `CLOSE`    |

---

## ✅ **Deliverables**

Each learner should:

1. Run all 9 steps in `psql`.
2. Capture screenshots of:

   * Variable, IF/ELSE, Loop outputs.
   * Function and procedure execution results.
   * Cursor iteration and safe divide function.
3. Submit a `.sql` file or screenshots with output evidence.

---

## 🧩 **Practice Challenges**

1. Write a function `hr.get_employee_bonus(dept TEXT)` that returns total bonus per department.
2. Create a procedure that reduces salary by 10 % for employees with rating < 3.
3. Implement a cursor to print all departments and count employees in each.
4. Add error handling to skip invalid salary records (< 0) and log them in a table.

---
