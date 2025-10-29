
# Lab 02: Run PostgreSQL using Docker

## 🎯 Objectives

By the end you will:

* Run PostgreSQL in Docker with persistent storage.
* Connect via `psql` and pgAdmin (optional).
* Use `docker compose` for a tidy, reproducible setup.
* Do logical backup/restore (`pg_dump`/`pg_restore`).
* Understand networking, health checks, and upgrades.

## 🧰 Prerequisites

* **Docker Desktop** (Windows/macOS) or **Docker Engine** (Linux) installed and running.
* Terminal/PowerShell access with basic Docker commands.
* ~1.5 GB free disk.

---

## Part 1 — Quick Start (ephemeral)

> Good for a rapid test; data will vanish when the container is removed.

```bash
docker run -d --name pg-temp \
  -e POSTGRES_PASSWORD=StrongP@ss_123 \
  -p 5432:5432 \
  postgres:16
```

Verify:

```bash
docker logs pg-temp --tail=50
docker exec -it pg-temp psql -U postgres -c "SELECT version();"
```

Stop/remove:

```bash
docker rm -f pg-temp
```

---

## Part 2 — Persistent PostgreSQL (named volume)

> Use a **named volume** so data survives container recreations.

1. Create volume and run:

```bash
docker volume create pgdata16
docker run -d --name pg16 \
  -e POSTGRES_PASSWORD=StrongP@ss_123 \
  -e POSTGRES_DB=trainingdb \
  -p 5432:5432 \
  -v pgdata16:/var/lib/postgresql/data \
  --health-cmd="pg_isready -U postgres || exit 1" \
  --health-interval=10s --health-timeout=5s --health-retries=5 \
  postgres:16
```

2. Check health & basic queries:

```bash
docker ps --filter "name=pg16"
docker inspect --format='{{json .State.Health}}' pg16 | jq .
docker exec -it pg16 psql -U postgres -d trainingdb -c "CREATE TABLE t(x int); INSERT INTO t VALUES (1); SELECT * FROM t;"
```

---

## Part 3 — Host-path persistence (alternative)

> Keeps data in a visible folder on your disk.

* **Windows (PowerShell, default Linux containers):**

  ```powershell
  mkdir C:\pgdata16
  docker run -d --name pg16-hp `
    -e POSTGRES_PASSWORD=StrongP@ss_123 `
    -e POSTGRES_DB=trainingdb `
    -p 5433:5432 `
    -v C:\pgdata16:/var/lib/postgresql/data `
    postgres:16
  ```

* **macOS/Linux (bash):**

  ```bash
  mkdir -p $HOME/pgdata16 && chmod 700 $HOME/pgdata16
  docker run -d --name pg16-hp \
    -e POSTGRES_PASSWORD=StrongP@ss_123 \
    -e POSTGRES_DB=trainingdb \
    -p 5433:5432 \
    -v $HOME/pgdata16:/var/lib/postgresql/data \
    postgres:16
  ```

Test:

```bash
psql -h localhost -p 5433 -U postgres -d trainingdb -c "SELECT current_database(), current_user;"
```

---

## Part 4 — Use docker compose (Postgres + pgAdmin)

Create a folder (e.g., `pg-lab`) and a file `compose.yaml`:

```yaml
services:
  db:
    image: postgres:16
    container_name: pg16_compose
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: StrongP@ss_123
      POSTGRES_DB: trainingdb
    ports:
      - "5432:5432"
    volumes:
      - pgdata16:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5

  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin4
    depends_on:
      db:
        condition: service_healthy
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: Admin@123
    ports:
      - "8080:80"
    volumes:
      - pgadmin_data:/var/lib/pgadmin

volumes:
  pgdata16:
  pgadmin_data:
```

Start:

```bash
docker compose up -d
docker compose ps
```

Open **pgAdmin** in your browser: `http://localhost:8080`

* Login: `admin@example.com` / `Admin@123`
* Add a new server:

  * **Name:** Local
  * **Connection → Host:** `db` *(Docker network name of the service)*
  * **Port:** `5432`
  * **Username:** `postgres`, **Password:** `StrongP@ss_123`

Test query in pgAdmin Query Tool:

```sql
SELECT version();
CREATE TABLE demo(id serial primary key, note text);
INSERT INTO demo(note) VALUES ('hello from pgAdmin');
SELECT * FROM demo;
```

Stop stack:

```bash
docker compose down
# keep volumes
# or remove everything (CAUTION):
# docker compose down -v
```

---

## Part 5 — Create user & grants (inside container)

```bash
docker exec -it pg16_compose psql -U postgres -d trainingdb -c "CREATE USER trainee WITH PASSWORD 'Tr@inee123';"
docker exec -it pg16_compose psql -U postgres -d trainingdb -c "GRANT CONNECT ON DATABASE trainingdb TO trainee;"
docker exec -it pg16_compose psql -U postgres -d trainingdb -c "GRANT USAGE ON SCHEMA public TO trainee;"
docker exec -it pg16_compose psql -U postgres -d trainingdb -c "GRANT SELECT,INSERT,UPDATE,DELETE ON ALL TABLES IN SCHEMA public TO trainee;"
```

---

## Part 6 — Backup & Restore

**Backup (custom format)**

```bash
# backup trainingdb into a file on your host Desktop (Linux/macOS example path)
docker exec -t pg16_compose pg_dump -U postgres -d trainingdb -F c > trainingdb.dump
```

**Restore (to a new DB)**

```bash
docker exec -it pg16_compose createdb -U postgres trainingdb_restore
docker exec -i pg16_compose pg_restore -U postgres -d trainingdb_restore < trainingdb.dump
```

> If you prefer storing dumps in a host folder, mount another bind:
> `-v $PWD/backups:/backups` and write to `/backups/trainingdb.dump` inside container.

---

## Part 7 — Networking patterns

* **From host to container:** use `localhost:5432` (or mapped port).
* **Container-to-container:** use the **service name** (`db`) on the **default compose network**, port `5432`.
* **Custom bridge (single `docker run` flow):**

  ```bash
  docker network create pgnet
  docker run -d --name pg16net --network pgnet -e POSTGRES_PASSWORD=pass -v pgdata16:/var/lib/postgresql/data postgres:16
  docker run -it --rm --network pgnet postgres:16 psql -h pg16net -U postgres -c "SELECT version();"
  ```

---

## Part 8 — Upgrading Postgres container (minor/major)

> **Minor** upgrades (e.g., 16.1 → 16.2): just recreate container with newer tag; volume keeps data.

```bash
docker pull postgres:16
docker compose up -d --pull always
```

> **Major** upgrades (e.g., 15 → 16) require **pg_upgrade** or dump/restore.

* Easiest path for small DBs: `pg_dump` on old major → start new major → `pg_restore`.
* For large DBs, use `pg_upgrade` (dedicated guide; plan downtime).

---

## Part 9 — Useful environment variables

* `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`
* Set `TZ` for timezone (optional):

  ```yaml
  environment:
    TZ: "Asia/Kolkata"
  ```
* Custom configs:

  * Mount a `.conf`:

    ```yaml
    volumes:
      - ./postgresql.conf:/etc/postgresql/postgresql.conf:ro
    command: ["postgres", "-c", "config_file=/etc/postgresql/postgresql.conf"]
    ```

---

## Part 10 — Troubleshooting

| Issue                          | Cause                                   | Fix                                                                                    |
| ------------------------------ | --------------------------------------- | -------------------------------------------------------------------------------------- |
| `port is already allocated`    | Host port 5432 in use                   | Change mapping to `-p 5433:5432` or stop the other Postgres.                           |
| Container restarts / exits     | Bad env / corrupted volume              | Check `docker logs <name>`. Recreate with clean env; verify volume path & permissions. |
| `psql: could not connect`      | Not healthy yet / firewall / wrong host | Wait for health, use correct host/port, ensure no VPN/firewall blocks.                 |
| Permission denied on host-path | Host folder perms                       | `chmod 700 <folder>` (Linux/macOS). On Windows, use a path shared with Docker Desktop. |
| pgAdmin can’t connect to `db`  | Wrong host in pgAdmin                   | Inside compose, host should be **`db`** (service name), not `localhost`.               |

---

