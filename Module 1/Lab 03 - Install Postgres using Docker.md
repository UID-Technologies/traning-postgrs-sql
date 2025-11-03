# Lab 03: Run PostgreSQL using Docker

## 🧩 Step 1: Pull PostgreSQL Image

Open **PowerShell** or **Command Prompt** and run:

```bash
docker pull postgres
```

This downloads the latest PostgreSQL image from Docker Hub.

---

## 🏗️ Step 2: Create and Run PostgreSQL Container

Run the following command to start a new container:

```bash
docker run --name postgres-container -e POSTGRES_PASSWORD=admin123 -p 5432:5432 -d postgres
```

**Explanation:**

* `--name postgres-container` → name of your container
* `-e POSTGRES_PASSWORD=admin123` → sets password for default user `postgres`
* `-p 5432:5432` → maps container port 5432 to host port 5432
* `-d` → runs container in detached mode
* `postgres` → image name

---

## 🔍 Step 3: Verify Container is Running

```bash
docker ps
```

You should see a line like:

```
CONTAINER ID   IMAGE      COMMAND                  STATUS          PORTS
abcd1234       postgres   "docker-entrypoint.s…"   Up 5 minutes    0.0.0.0:5432->5432/tcp
```

---

## 🧠 Step 4: Get Inside the PostgreSQL Container

Run this command to open a shell inside the container:

```bash
docker exec -it postgres-container bash
```

Now you’ll be inside the container terminal — the prompt will change to something like:

```
root@abcd1234:/#
```

---

## 🐘 Step 5: Access PostgreSQL CLI (psql)

Once inside the container, connect to PostgreSQL:

```bash
psql -U postgres
```

Enter the password you set (`admin123`), and you’ll get the PostgreSQL prompt:

```
postgres=#
```

---

## ✅ Step 6: Test Database Commands

Try running:

```sql
CREATE DATABASE companydb;
\l
\q
```

---

## 🧹 Step 7: Stop and Start Container (Optional)

To stop:

```bash
docker stop postgres-container
```

To start again:

```bash
docker start postgres-container
```

---

Would you like me to show a **docker-compose.yml** version for PostgreSQL (for easier re-use)?
