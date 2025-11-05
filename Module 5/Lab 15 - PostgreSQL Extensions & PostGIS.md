# 🧩 **Lab 15 – PostgreSQL Extensions, PostGIS & Capstone Project (Hands-on Lab)**

⏱️ *Duration: 5 Hours*

---

## 🎯 **Objectives**

By the end of this lab, learners will:

* Work with **JSON, JSONB, Arrays, and hstore** data types.
* Install and configure **PostGIS** (spatial extension).
* Explore **spatial data types** (`geometry`, `geography`) and queries.
* Execute **GIS operations** like radius, polygon, and proximity searches.
* Configure PostgreSQL in **Docker** for scalability.
* Implement **tuning parameters** (`shared_buffers`, `work_mem`, etc.).
* Build a **Geospatial Employee Distribution & Reporting System** (Capstone).

---

## 🧠 **Concept Overview**

| Topic                  | Description                                                                    |
| ---------------------- | ------------------------------------------------------------------------------ |
| **JSON / JSONB**       | Store semi-structured data (JSONB is binary-optimized for indexing).           |
| **Arrays & hstore**    | Store lists or key–value pairs inside a single column.                         |
| **PostGIS**            | Spatial extension adding geographic and geometric data support.                |
| **Spatial Types**      | `geometry`, `geography`, `POINT`, `POLYGON`, etc.                              |
| **GIS Queries**        | Functions like `ST_Distance`, `ST_Within`, `ST_Intersects`, etc.               |
| **Scaling Parameters** | Tune performance via `shared_buffers`, `work_mem`, and `maintenance_work_mem`. |

---

## 🧰 **Setup**

Start a PostgreSQL + PostGIS container:

```bash
docker run -d --name pg-postgis \
  -e POSTGRES_PASSWORD=Admin@123 \
  -p 5432:5432 \
  postgis/postgis:15-3.4
```

Access container and psql:

```bash
docker exec -it pg-postgis bash
psql -U postgres
```

Create a lab database:

```sql
CREATE DATABASE postgis_lab;
\c postgis_lab;
```

Enable required extensions:

```sql
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS hstore;
```

✅ **Observation:**
PostGIS and hstore are now active in this database.

---

# 🧪 **Lab Exercises**

---

## 🔹 **Step 1 – JSON & JSONB Data Types**

Create a table to store employee details:

```sql
CREATE TABLE employee_json (
  id SERIAL PRIMARY KEY,
  name TEXT,
  details JSONB
);
```

Insert JSON data:

```sql
INSERT INTO employee_json (name, details) VALUES
('Asha', '{"dept": "IT", "skills": ["Python", "SQL"], "active": true}'),
('Rohit', '{"dept": "Finance", "skills": ["Excel", "PowerBI"], "active": false}');
```

Query JSON fields:

```sql
SELECT name, details->>'dept' AS department FROM employee_json;
SELECT * FROM employee_json WHERE details->>'active' = 'true';
```

✅ **Observation:**
JSONB fields can be queried efficiently using operators like `->` and `->>`.

---

## 🔹 **Step 2 – Arrays and hstore**

```sql
CREATE TABLE employee_arrays (
  id SERIAL PRIMARY KEY,
  name TEXT,
  projects TEXT[],
  attributes hstore
);

INSERT INTO employee_arrays (name, projects, attributes) VALUES
('Pooja', ARRAY['Alpha', 'Beta'], 'role => HR, level => Senior'),
('Ravi', ARRAY['Gamma'], 'role => Dev, level => Mid');

SELECT name, projects[1] AS first_project, attributes->'role' AS role
FROM employee_arrays;
```

✅ **Observation:**
Arrays and hstore store multiple values compactly, ideal for metadata or dynamic tags.

---

## 🔹 **Step 3 – PostGIS Spatial Basics**

Create a spatial table:

```sql
CREATE TABLE office_locations (
  id SERIAL PRIMARY KEY,
  office_name TEXT,
  location GEOGRAPHY(POINT)
);
```

Insert sample coordinates (latitude, longitude):

```sql
INSERT INTO office_locations (office_name, location) VALUES
('Delhi HQ', ST_GeogFromText('POINT(77.2090 28.6139)')),
('Mumbai Office', ST_GeogFromText('POINT(72.8777 19.0760)')),
('Bangalore Branch', ST_GeogFromText('POINT(77.5946 12.9716)'));
```

---

## 🔹 **Step 4 – GIS Queries**

Find offices within a 500 km radius of Delhi:

```sql
SELECT office_name,
       ST_Distance(location, ST_GeogFromText('POINT(77.2090 28.6139)')) AS distance_km
FROM office_locations
WHERE ST_DWithin(location, ST_GeogFromText('POINT(77.2090 28.6139)'), 500000);
```

✅ **Observation:**
`ST_DWithin` helps locate points within a distance threshold.

Find which offices fall inside a polygon boundary:

```sql
WITH region AS (
  SELECT ST_GeogFromText('POLYGON((72 18, 73 18, 73 20, 72 20, 72 18))') AS boundary
)
SELECT o.office_name
FROM office_locations o, region
WHERE ST_Within(o.location, region.boundary);
```

---

## 🔹 **Step 5 – PostgreSQL Configuration Tuning**

```sql
SHOW shared_buffers;
SHOW work_mem;
SHOW maintenance_work_mem;

ALTER SYSTEM SET shared_buffers = '512MB';
ALTER SYSTEM SET work_mem = '32MB';
ALTER SYSTEM SET maintenance_work_mem = '128MB';
SELECT pg_reload_conf();
```

✅ **Observation:**
These parameters help optimize performance for large datasets or analytical workloads.

---

## 🔹 **Step 6 – Materialized Views & Indexing**

Create a summary view:

```sql
CREATE MATERIALIZED VIEW emp_summary AS
SELECT details->>'dept' AS dept, COUNT(*) AS total
FROM employee_json
GROUP BY dept;

CREATE INDEX idx_json_dept ON employee_json USING GIN (details);
REFRESH MATERIALIZED VIEW emp_summary;
SELECT * FROM emp_summary;
```

✅ **Observation:**
Materialized views cache aggregated data and GIN indexes speed up JSONB queries.

---

# 🧭 **Capstone Project: Geospatial Employee Distribution & Reporting System**

---

## 🧩 **Objective**

Design a system to analyze employee distribution across cities using **PostGIS** and **JSONB** for flexible attributes.

---

## 🔹 **Step 1 – Schema Design**

```sql
CREATE TABLE departments (
  dept_id SERIAL PRIMARY KEY,
  dept_name TEXT UNIQUE
);

CREATE TABLE employees (
  emp_id SERIAL PRIMARY KEY,
  full_name TEXT,
  dept_id INT REFERENCES departments(dept_id),
  meta JSONB,
  location GEOGRAPHY(POINT)
);
```

---

## 🔹 **Step 2 – Load Sample Data**

```sql
INSERT INTO departments (dept_name) VALUES ('IT'), ('Finance'), ('HR');

INSERT INTO employees (full_name, dept_id, meta, location) VALUES
('Asha Singh', 1, '{"skills":["SQL","Python"],"level":"Senior"}', ST_GeogFromText('POINT(77.2090 28.6139)')),
('Rohit Verma', 2, '{"skills":["Excel"],"level":"Mid"}', ST_GeogFromText('POINT(72.8777 19.0760)')),
('Pooja Das', 3, '{"skills":["Recruitment"],"level":"Junior"}', ST_GeogFromText('POINT(77.5946 12.9716)'));
```

---

## 🔹 **Step 3 – Heatmap Query: Region-wise Distribution**

```sql
SELECT d.dept_name,
       COUNT(e.emp_id) AS emp_count,
       ST_AsText(ST_Centroid(ST_Collect(e.location))) AS region_center
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id
GROUP BY d.dept_name;
```

✅ **Observation:**
You can plot centroid coordinates for a **region heatmap** in visualization tools like QGIS or Power BI.

---

## 🔹 **Step 4 – Indexing and Materialized View**

```sql
CREATE INDEX idx_emp_location ON employees USING GIST (location);
CREATE INDEX idx_emp_meta ON employees USING GIN (meta);

CREATE MATERIALIZED VIEW emp_geo_summary AS
SELECT dept_id, COUNT(*) AS emp_count
FROM employees
GROUP BY dept_id;
REFRESH MATERIALIZED VIEW emp_geo_summary;
```

---

## 🔹 **Step 5 – Security & Auditing**

```sql
CREATE ROLE hr_analyst LOGIN PASSWORD 'Hr@123';
GRANT CONNECT ON DATABASE postgis_lab TO hr_analyst;
GRANT SELECT ON employees, departments TO hr_analyst;

ALTER SYSTEM SET log_statement = 'ddl';
SELECT pg_reload_conf();
```

✅ **Observation:**
Only authorized roles can access the data; all DDL statements are logged.

---

## 🧾 **Summary Table**

| Area            | Concept                   | Example                   |
| --------------- | ------------------------- | ------------------------- |
| JSON / JSONB    | Semi-structured data      | `details->>'dept'`        |
| hstore / Arrays | Key-value & lists         | `attributes->'role'`      |
| Spatial Data    | Geolocation storage       | `GEOGRAPHY(POINT)`        |
| GIS Queries     | Radius & polygon checks   | `ST_DWithin`, `ST_Within` |
| Performance     | Index + Materialized View | `GIN`, `GIST`             |
| Security        | Roles & auditing          | `log_statement='ddl'`     |

---

## ✅ **Deliverables**

Each learner must:

1. Capture JSON, PostGIS, and heatmap query results.
2. Show GIN/GIST index creation.
3. Display region-wise employee count query output.
4. Submit `emp_geo_summary` output and sample audit log entry.

---

## 🧩 **Practice Challenges**

1. Create a new department and visualize its geographic spread.
2. Use `ST_Buffer` to find all employees within a 10 km radius of HQ.
3. Store employee home coordinates and compute commute distance.
4. Automate materialized view refresh via cron or pgAgent.
5. Tune configuration parameters for large-scale datasets.

---
