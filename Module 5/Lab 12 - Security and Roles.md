
# 🧩 **Lab 12 – PostgreSQL Security, Access Control & Auditing (Hands-on Lab)**

---

## 🎯 **Objectives**

By the end of this lab, learners will:

* Understand PostgreSQL **Role-Based Access Control (RBAC)**
* Create and manage **roles and privileges** using `GRANT` and `REVOKE`
* Configure **schema-level and object-level access**
* Learn about **authentication methods** (`md5`, `scram-sha-256`)
* Implement **Row-Level Security (RLS)** policies
* Enable **auditing and logging** for compliance
* Practice **encryption** at rest and in transit

---

## 🧠 **Concept Overview**

| Topic              | Description                                                                   |
| ------------------ | ----------------------------------------------------------------------------- |
| **RBAC**           | Assign privileges based on roles rather than individual users                 |
| **GRANT / REVOKE** | Allow or remove access to schemas, tables, and objects                        |
| **RLS**            | Control which rows a user can view or modify                                  |
| **Auditing**       | Record statements, logins, and activity via PostgreSQL settings or extensions |
| **Encryption**     | Protect data in storage and during transmission                               |

---

## 🧰 **Setup**

Start with your running PostgreSQL container (from previous labs):

```bash
docker exec -it postgres-container bash
psql -U postgres
```

Create a dedicated database for this lab:

```sql
CREATE DATABASE securitylab;
\c securitylab;
```

---

# 🧪 **Lab Exercises**

---

## 🔹 **Step 1 – Create Roles and Users (RBAC)**

### Goal

Define three key roles: `admin_role`, `dev_role`, and `readonly_role`.

```sql
-- 1️⃣ Create roles
CREATE ROLE admin_role NOINHERIT;
CREATE ROLE dev_role NOINHERIT;
CREATE ROLE readonly_role NOINHERIT;

-- 2️⃣ Create users and assign roles
CREATE USER admin_user WITH PASSWORD 'Admin@123';
CREATE USER dev_user WITH PASSWORD 'Dev@123';
CREATE USER readonly_user WITH PASSWORD 'Readonly@123';

-- 3️⃣ Grant roles to users
GRANT admin_role TO admin_user;
GRANT dev_role TO dev_user;
GRANT readonly_role TO readonly_user;
```

✅ **Observation:**
Each user now belongs to a role with specific privilege boundaries.

---

## 🔹 **Step 2 – Create a Schema and Base Table**

```sql
CREATE SCHEMA corp AUTHORIZATION admin_user;

CREATE TABLE corp.employee (
  emp_id SERIAL PRIMARY KEY,
  full_name TEXT,
  salary NUMERIC(10,2),
  dept TEXT
);

INSERT INTO corp.employee (full_name, salary, dept) VALUES
('Asha Singh', 1200000, 'IT'),
('Rohit Verma', 950000, 'Finance'),
('Pooja Das', 600000, 'HR');
```

---

## 🔹 **Step 3 – Grant / Revoke Privileges**

### Example Privilege Setup

```sql
-- Admin can do everything
GRANT ALL PRIVILEGES ON SCHEMA corp TO admin_role;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA corp TO admin_role;

-- Dev can read and write data
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA corp TO dev_role;

-- Readonly can only query
GRANT SELECT ON ALL TABLES IN SCHEMA corp TO readonly_role;

-- Revoke accidental access
REVOKE DELETE ON ALL TABLES IN SCHEMA corp FROM dev_role;
```

✅ **Observation:**
Privileges are assigned based on responsibilities — enforcing least privilege principle.

---

## 🔹 **Step 4 – Schema & Object Access Rules**

### Restrict schema visibility for non-admins

```sql
REVOKE USAGE ON SCHEMA corp FROM PUBLIC;
GRANT USAGE ON SCHEMA corp TO admin_role, dev_role, readonly_role;
```

✅ **Observation:**
Only assigned roles can access schema objects; others cannot even list the schema.

---

## 🔹 **Step 5 – Authentication Methods Overview**

> Authentication is configured in **`pg_hba.conf`**, located in the container’s `/var/lib/postgresql/data`.

| Method          | Description                           |
| --------------- | ------------------------------------- |
| `trust`         | No password (unsafe for production)   |
| `md5`           | MD5 password hashing                  |
| `scram-sha-256` | Modern, stronger password hashing     |
| `peer`          | System user match (local connections) |

### Check your current method:

```bash
cat /var/lib/postgresql/data/pg_hba.conf | grep -v '^#'
```

To switch to **SCRAM authentication**:

```bash
echo "host all all all scram-sha-256" >> /var/lib/postgresql/data/pg_hba.conf
```

Then restart PostgreSQL inside container:

```bash
pg_ctl -D /var/lib/postgresql/data restart
```

✅ **Observation:**
SCRAM is more secure and now used for password authentication.

---

## 🔹 **Step 6 – Row-Level Security (RLS)**

### Goal

Ensure each user can only view their own department’s data.

```sql
-- Enable RLS
ALTER TABLE corp.employee ENABLE ROW LEVEL SECURITY;

-- Policy for IT department
CREATE POLICY it_policy ON corp.employee
FOR SELECT USING (current_user = 'dev_user' AND dept = 'IT');

-- Policy for readonly_user (HR only)
CREATE POLICY hr_policy ON corp.employee
FOR SELECT USING (current_user = 'readonly_user' AND dept = 'HR');
```

Test as `dev_user`:

```sql
SET ROLE dev_user;
SELECT * FROM corp.employee;
RESET ROLE;
```

✅ **Observation:**
`dev_user` only sees IT employees; others are hidden.

---

## 🔹 **Step 7 – Auditing & Logging**

### 1️⃣ Enable Statement Logging (inside container)

Open `postgresql.conf`:

```bash
cat >> /var/lib/postgresql/data/postgresql.conf <<EOF
log_statement = 'all'
log_connections = on
log_disconnections = on
EOF
```

Restart service:

```bash
pg_ctl -D /var/lib/postgresql/data restart
```

✅ **Observation:**
Logs are written to `/var/lib/postgresql/data/log/` (or `/var/log/postgresql`).

---

### 2️⃣ Install and Use `pgaudit` (if available)

If the image supports it (Postgres 13+):

```sql
CREATE EXTENSION pgaudit;
ALTER SYSTEM SET pgaudit.log = 'all';
SELECT * FROM pg_available_extensions WHERE name='pgaudit';
```

✅ **Observation:**
Audits will now capture DDL/DML activity.

---

## 🔹 **Step 8 – Encryption (Data-at-Rest)**

### Using `pgcrypto` for column encryption

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Add encrypted column
ALTER TABLE corp.employee ADD COLUMN ssn BYTEA;

-- Encrypt sensitive data
UPDATE corp.employee
SET ssn = pgp_sym_encrypt('123-45-6789', 'SecretKey');

-- Decrypt to view
SELECT emp_id, full_name,
       convert_from(pgp_sym_decrypt(ssn, 'SecretKey'), 'UTF8') AS decrypted_ssn
FROM corp.employee;
```

✅ **Observation:**
SSNs stored as ciphertext in table; decrypted only when needed.

---

## 🔹 **Step 9 – Encryption (Data-in-Transit)**

### Enable SSL/TLS in PostgreSQL

Inside container:

```bash
cd /var/lib/postgresql/data
openssl req -new -x509 -days 365 -nodes \
  -text -out server.crt -keyout server.key -subj "/CN=postgres"
chmod 600 server.key
```

Edit `postgresql.conf`:

```bash
echo "ssl = on" >> /var/lib/postgresql/data/postgresql.conf
pg_ctl -D /var/lib/postgresql/data restart
```

From client:

```bash
psql "sslmode=require host=localhost dbname=securitylab user=postgres password=Admin@123"
```

✅ **Observation:**
Connection established securely over SSL.

---

## 🧾 **Summary Table**

| Area                 | Concept             | Command / File             |
| -------------------- | ------------------- | -------------------------- |
| RBAC                 | Role creation       | `CREATE ROLE`, `GRANT`     |
| Privileges           | Fine-grained access | `GRANT / REVOKE`           |
| Authentication       | Password hashing    | `md5`, `scram-sha-256`     |
| RLS                  | Per-user filters    | `CREATE POLICY`            |
| Auditing             | Activity logs       | `log_statement`, `pgaudit` |
| Encryption (rest)    | Column encryption   | `pgcrypto`                 |
| Encryption (transit) | SSL/TLS setup       | `ssl = on`                 |

---

## ✅ **Deliverables**

Each learner should:

1. Create and configure three roles (`admin`, `dev`, `readonly`).
2. Demonstrate RBAC with `GRANT`/`REVOKE`.
3. Enable and test **RLS** and **logging**.
4. Show **pgcrypto encryption** and **SSL-enabled connection**.
5. Submit screenshots of:

   * Role creation and privileges
   * RLS query results
   * Encrypted/decrypted data
   * SSL connection status

---

## 🧩 **Practice Challenges**

1. Create a custom RLS policy where a manager can see all employees in their department.
2. Revoke all write permissions from `readonly_user` and verify enforcement.
3. Encrypt multiple fields (like `salary`, `email`) using different keys.
4. Enable `log_duration` and analyze query execution times.
5. Use an external SSL certificate for production-like configuration.

---

