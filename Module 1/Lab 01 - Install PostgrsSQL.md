# Lab 01: Install PostgreSQL on Windows (with pgAdmin & psql)

## 🎯 Objectives

By the end, you will:

* Install PostgreSQL (latest stable) on Windows.
* Set up a superuser, data directory, and Windows service.
* Verify using **pgAdmin** GUI and **psql** CLI.
* Create a database, user, and table.
* (Optional) Install via Chocolatey.
* Learn basic backup/restore and common fixes.

---

## 🧰 Prerequisites

* Windows 10/11 (64-bit), local admin rights.
* ~1.5 GB free disk space.
* Internet access.
* If corporate laptop: confirm installer execution isn’t blocked by policy.

---

## Part 1 — Download & Launch Installer (EDB One-Click)

1. **Download**

   * Go to the official PostgreSQL download page.
   * Choose **Windows → PostgreSQL Version (e.g., 16/17)** → download the **EDB installer (.exe)**.

2. **Run as Administrator**

   * Right-click the `.exe` → **Run as administrator**.
   * If SmartScreen prompts, choose **More info → Run anyway** (if applicable).

---

## Part 2 — Guided Installation

1. **Welcome Screen**

   * Click **Next**.

2. **Installation Directory**

   * Default is `C:\Program Files\PostgreSQL\<version>\` → **Next**.

3. **Select Components**

   * Keep defaults checked:

     * **PostgreSQL Server**
     * **pgAdmin 4**
     * **Command Line Tools** (psql, pg_dump, etc.)
     * **Stack Builder** *(optional; we’ll skip now)*
   * **Next**.

4. **Data Directory**

   * Default: `C:\Program Files\PostgreSQL\<version>\data`
   * For production/dev boxes, you can use a separate drive (e.g., `D:\pgdata`) → **Next**.

5. **Database Superuser Password**

   * Username is **postgres** (superuser).
   * Enter a strong password (save it!) → **Next**.

6. **Port**

   * Default: **5432** → **Next**.
   * If port conflict is likely, pick another (e.g., 5433). Note it down.

7. **Locale**

   * Default is fine; otherwise choose your system locale → **Next**.

8. **Pre-Installation Summary**

   * Review → **Next** to install. Wait for completion.

9. **Stack Builder**

   * Uncheck **Stack Builder** for now → **Finish**.

> ✅ **Checkpoint A:** PostgreSQL service should be installed and running as a Windows service named **postgresql-x64-<version>**.

---

## Part 3 — Verify the Service & Environment

1. **Check Windows Service**

   * Press **Win + R** → type `services.msc` → Enter.
   * Find **postgresql-x64-<version>** → Status should be **Running**.
   * If not running: right-click → **Start** (see Troubleshooting section if it fails).

2. **Add psql to PATH (if needed)**

   * The installer usually sets PATH. If `psql` isn’t found later:

     * Start → search **“Edit the system environment variables”** → **Environment Variables**.
     * Under **System variables** → select **Path** → **Edit** → **New**:

       ```
       C:\Program Files\PostgreSQL\<version>\bin
       ```
     * **OK** all dialogs.

---

## Part 4 — Verify Using pgAdmin (GUI)

1. **Open pgAdmin 4**

   * Start → **pgAdmin 4**.
   * First launch creates a local profile; you may be asked to set a **pgAdmin master password** (this secures saved connections).

2. **Connect to Local Server**

   * In the left **Browser**, expand **Servers** → **PostgreSQL <version>**.
   * If it asks for a password, enter the **postgres** password you set during install.
   * Expand **Databases** → you should see **postgres**, **template0**, **template1**.

> ✅ **Checkpoint B:** You can view server properties and databases in pgAdmin.

---

## Part 5 — Verify Using psql (CLI)

1. **Open Command Prompt or PowerShell**

2. **Connect:**

   ```bat
   psql -U postgres -h localhost -p 5432
   ```

   * Enter the password when prompted.

3. **Run Basics:**

   ```sql
   \l          -- list databases
   \conninfo   -- show connection info
   SELECT version();
   \q          -- quit
   ```

> ✅ **Checkpoint C:** You connected with psql and executed commands.

---

## Part 6 — First Admin Tasks (DB, Role, Table)

1. **Create a Database & User (via psql or pgAdmin → Query Tool)**

   ```sql
   -- Connect as postgres superuser first
   CREATE DATABASE trainingdb;

   -- Create a limited user
   CREATE USER trainee WITH PASSWORD 'StrongP@ss_123';

   -- Grant minimal rights
   GRANT CONNECT ON DATABASE trainingdb TO trainee;
   ```

2. **Connect to trainingdb and Create Objects**

   ```bat
   psql -U postgres -d trainingdb -h localhost
   ```

   ```sql
   CREATE SCHEMA app AUTHORIZATION postgres;

   CREATE TABLE app.student (
     student_id   SERIAL PRIMARY KEY,
     name         VARCHAR(50) NOT NULL,
     age          INT CHECK (age >= 18),
     created_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );

   INSERT INTO app.student (name, age) VALUES
     ('Varun Gupta', 25),
     ('Neha Sharma', 22);

   SELECT * FROM app.student;
   ```

3. **Grant Access to trainee**

   ```sql
   GRANT USAGE ON SCHEMA app TO trainee;
   GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA app TO trainee;
   ALTER DEFAULT PRIVILEGES IN SCHEMA app GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO trainee;
   ```

> ✅ **Checkpoint D:** You created a DB, a user, and verified data insertion.

---

## Part 7 — Optional: Install via Chocolatey (One-Liners)

If you use **Chocolatey** (must be installed already, run Admin PowerShell):

```powershell
choco install postgresql16 --params '/Password:StrongP@ss_123' -y
# or for latest major (adjust version as needed)
# choco install postgresql --pre -y
```

> After install, ensure `C:\Program Files\PostgreSQL\<version>\bin` is in PATH, then use `psql`.

---

## Part 8 — Basic Backup & Restore (Windows)

1. **Logical Backup (Custom Format)**

   ```bat
   pg_dump -U postgres -d trainingdb -F c -f "%USERPROFILE%\Desktop\trainingdb.dump"
   ```

2. **Restore to a New DB**

   ```bat
   createdb -U postgres trainingdb_restore
   pg_restore -U postgres -d trainingdb_restore "%USERPROFILE%\Desktop\trainingdb.dump"
   ```

---

## Part 9 — Common Configuration Tweaks

1. **Change Port or Listen Address**

   * Edit: `C:\Program Files\PostgreSQL\<version>\data\postgresql.conf`

     ```conf
     port = 5432
     listen_addresses = 'localhost'   # or '0.0.0.0' for remote access (with firewall + pg_hba.conf)
     ```
   * Edit client access in `pg_hba.conf` (same folder). Example to allow local IPv4:

     ```
     host    all    all    127.0.0.1/32    scram-sha-256
     ```
   * **Restart service** after changes:

     * Services → select PostgreSQL service → **Restart**.

2. **Firewall**

   * If allowing remote access, open the chosen port (default 5432) in Windows Defender Firewall (Inbound rule).

---

## Part 10 — Troubleshooting

| Symptom                            | Likely Cause                              | Fix                                                                                                                                                               |
| ---------------------------------- | ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Service won’t start                | Port in use / corrupt data dir            | Change `port` in `postgresql.conf`, or ensure only one instance uses the data directory. Check Windows Event Viewer → **Windows Logs → Application** for details. |
| `psql` not recognized              | PATH not set                              | Add `...PostgreSQL\<version>\bin` to **Path** (System variables). Reopen terminal.                                                                                |
| `password authentication failed`   | Wrong password / method mismatch          | Confirm user/password, check `pg_hba.conf` method (`md5`/`scram-sha-256`), reload/restart.                                                                        |
| Can’t connect remotely             | `listen_addresses`/`pg_hba.conf`/Firewall | Set `listen_addresses='0.0.0.0'`, allow in `pg_hba.conf`, open firewall, restart service.                                                                         |
| pgAdmin asks for “master password” | pgAdmin vault feature                     | Set/remember this (it’s separate from DB password). You can disable keyring/vault in pgAdmin settings if needed.                                                  |

---

## Cleanup / Uninstall (if needed)

* **Apps & Features** → **PostgreSQL <version>** → **Uninstall**.
* Optionally remove `data` directory (backup first if you need data).
* Remove PATH entry if you added it manually.

---

## Deliverables (for learners)

* Screenshot of **pgAdmin** connected to local server.
* `\l` output in **psql** showing `trainingdb`.
* Result of `SELECT * FROM app.student;`.
* A `.dump` file created on Desktop.

---

