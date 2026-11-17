# Practical 5: Docker Compose Multi-Container Application (Flask + MySQL)

> **Exam / Lab Weightage:** 30 Marks  
> **Aim:** Run a Python (Flask) web application and a MySQL database as two interconnected containers using Docker Compose, and demonstrate data persistence and retrieval across services.  
> **Project Directory:** `compose-app`  
> **Application URL:** `http://localhost:8000`

---

## Table of Contents
1. [Overview & Architecture](#overview--architecture)
2. [Step 1: Project Setup and Source Files (5 Marks)](#step-1-project-setup-and-source-files-5-marks)
3. [Step 2: Write `docker-compose.yml` (12 Marks)](#step-2-write-docker-composeyml-12-marks)
4. [Step 3: Start Services & Verify Container Health (5 Marks)](#step-3-start-services--verify-container-health-5-marks)
5. [Step 4: Verify Web App & Database Data (5 Marks)](#step-4-verify-web-app--database-data-5-marks)
6. [Step 5: Scaling, Logs, and Teardown with Volumes (3 Marks)](#step-5-scaling-logs-and-teardown-with-volumes-3-marks)
7. [Safe Customizations & Configuration Reference](#safe-customizations--configuration-reference)
8. [Troubleshooting Guide](#troubleshooting-guide)
9. [Screenshots Checklist](#screenshots-checklist)

---

## Overview & Architecture

Docker Compose manages multi-container Docker applications defined in a single file (`docker-compose.yml`):
- **Shared Bridge Network:** Docker Compose automatically creates a shared network (`compose-app_default`). Containers communicate using service names as hostnames; Flask connects to MySQL via `host="db"`.
- **Named Volume (`db_data`):** Persists MySQL data on the host machine (`/var/lib/mysql`) even if containers are stopped or removed.
- **Service Dependency & Health Check:** Flask (`web`) uses `depends_on` with `condition: service_healthy` so it only launches after MySQL passes its automated health check (`mysqladmin ping`).

```
+-------------------------------------------------------------+
|                      Host (Port 8000)                       |
+-------------------------------------------------------------+
                               |
                               v
+-----------------------+              +----------------------+
|  web (Flask App)      |  host="db"   |  db (MySQL 8.0)      |
|  Port 80 (Internal)   | -----------> |  Port 3306 (Internal)|
+-----------------------+              +----------------------+
                                                  |
                                                  v
                                       +----------------------+
                                       | Volume: db_data      |
                                       +----------------------+
```

---

## Step 1: Project Setup and Source Files (5 Marks)

In PowerShell, navigate to your workspace and create the project files:

```powershell
mkdir compose-app
cd compose-app
New-Item app.py, requirements.txt, Dockerfile, init.sql, docker-compose.yml -ItemType File
```

### File 1: `app.py` (Flask Web Application)
Connects to the `db` service, queries the `students` table, and returns a styled HTML table:

```python
from flask import Flask
import mysql.connector
import os

app = Flask(__name__)

@app.route("/")
def home():
    try:
        db = mysql.connector.connect(
            host="db",
            user="root",
            password=os.environ["MYSQL_ROOT_PASSWORD"],
            database=os.environ["MYSQL_DATABASE"]
        )
        cursor = db.cursor()
        cursor.execute("SELECT id, name, course FROM students")
        rows = cursor.fetchall()
        cursor.close()
        db.close()
        
        table_rows = "".join(f"<tr><td>{r[0]}</td><td>{r[1]}</td><td>{r[2]}</td></tr>" for r in rows)
        return f"""
        <html><body style="font-family:Arial;text-align:center;margin-top:60px">
        <h1>Docker Compose: Flask + MySQL</h1>
        <p style="color:green">Connected to MySQL successfully!</p>
        <table border="1" cellpadding="8" style="margin:auto">
        <tr><th>ID</th><th>Name</th><th>Course</th></tr>
        {table_rows}
        </table>
        </body></html>
        """
    except mysql.connector.Error as e:
        return f"Database connection failed: {e}", 503

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```

---

### File 2: `requirements.txt`
```text
flask
mysql-connector-python
```

---

### File 3: `Dockerfile`
Builds the Python application environment:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 80
CMD ["python", "app.py"]
```

---

### File 4: `init.sql`
Initializes database schema and seed data when MySQL starts for the first time:

```sql
CREATE TABLE IF NOT EXISTS students (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(50),
  course VARCHAR(50)
);

INSERT INTO students (name, course) VALUES
  ('Arvind', 'DevOps'),
  ('Priya', 'Data Science'),
  ('Rahul', 'Cloud Computing');
```

---

## Step 2: Write `docker-compose.yml` (12 Marks)

Create `docker-compose.yml` in the `compose-app` directory:

```yaml
services:
  web:
    build: .
    ports:
      - "8000:80"
    environment:
      MYSQL_ROOT_PASSWORD: rootpass123
      MYSQL_DATABASE: studentdb
    depends_on:
      db:
        condition: service_healthy
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass123
      MYSQL_DATABASE: studentdb
    volumes:
      - db_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h localhost -uroot -p$$MYSQL_ROOT_PASSWORD --silent"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s
volumes:
  db_data:
```

### Component Breakdown

| Configuration Directive | Purpose / Functionality |
| :--- | :--- |
| `web: build: .` | Builds the Flask image from the local `Dockerfile`. |
| `ports: "8000:80"` | Forwards traffic from host `http://localhost:8000` to container port `80`. |
| `depends_on: db: condition: service_healthy` | Ensures `web` starts only **after** the MySQL database passes its health check. |
| `db: image: mysql:8.0` | Uses official Oracle MySQL 8.0 image. |
| `MYSQL_ROOT_PASSWORD` & `MYSQL_DATABASE` | Sets root password (`rootpass123`) and default schema (`studentdb`). |
| `volumes: db_data:/var/lib/mysql` | Named volume ensuring database records persist across container restarts. |
| `./init.sql:/docker-entrypoint-initdb.d/...` | Mounts init SQL script executed once on initial database creation. |
| `healthcheck` | Periodically pings MySQL (`mysqladmin ping`) to verify readiness. |

### Validate the Configuration
Run in PowerShell:
```powershell
dir
docker compose config
```
- `dir` confirms all 5 files exist and have non-zero file sizes.
- `docker compose config` validates the YAML syntax and outputs the parsed service structure.

---

## Step 3: Start Services & Verify Container Health (5 Marks)

Start the multi-container stack in the background with image building:

```powershell
docker compose up -d --build
```

Wait ~20-30 seconds for MySQL to complete initialization and become healthy, then check status:

```powershell
docker compose ps
```

### Expected Output:
```text
NAME                IMAGE             COMMAND                  SERVICE   STATUS                        PORTS
compose-app-db-1    mysql:8.0         "docker-entrypoint.s…"   db        Up About a minute (healthy)   3306/tcp, 33060/tcp
compose-app-web-1   compose-app-web   "python app.py"          web       Up About a minute             0.0.0.0:8000->80/tcp
```

> **Screenshot 1:** Output of `docker compose ps` showing `compose-app-db-1` as `(healthy)` and `compose-app-web-1` as `Up`.

---

## Step 4: Verify Web App & Database Data (5 Marks)

### 1. Browser Verification
Open `http://localhost:8000` in Google Chrome or Edge.  
You should see:
- Heading: **Docker Compose: Flask + MySQL**
- Subtext: **Connected to MySQL successfully!** (in green)
- Table displaying initial database records:
  - `1 | Arvind | DevOps`
  - `2 | Priya | Data Science`
  - `3 | Rahul | Cloud Computing`

> **Screenshot 2:** Browser showing `http://localhost:8000` with the student database table.

---

### 2. Direct MySQL Query via CLI
Verify records directly inside the running database container:

```powershell
docker compose exec db mysql -uroot -prootpass123 -e "SELECT * FROM studentdb.students;"
```

### Expected Output:
```text
+----+--------+-----------------+
| id | name   | course          |
+----+--------+-----------------+
|  1 | Arvind | DevOps          |
|  2 | Priya  | Data Science    |
|  3 | Rahul  | Cloud Computing |
+----+--------+-----------------+
```

---

## Step 5: Scaling, Logs, and Teardown with Volumes (3 Marks)

### 1. Scaling the Web Service
Demonstrate service scaling down to 0 and back to 1:

```powershell
# Scale web to 0 (stops web container)
docker compose up -d --scale web=0
docker compose ps

# Scale web back to 1 (restarts web container)
docker compose up -d --scale web=1
docker compose ps
```
*(Note: Host port `8000` is bound to a single container, so web cannot scale above 1 without an external reverse proxy).*

### 2. Inspecting Service Logs
View live standard output and access logs:

```powershell
# Flask web server access logs
docker compose logs web

# MySQL database initialization logs
docker compose logs db
```

### 3. Teardown with Volumes Cleanup
Stop and remove containers, networks, and persistent storage volumes:

```powershell
docker compose down -v
docker volume ls
```
- `-v`: Deletes named volumes (`compose-app_db_data`), resetting database storage.
- `docker volume ls`: Confirms that the volume is removed.

> **Screenshot 3:** Terminal output showing `--scale web=0`, `--scale web=1`, `docker compose logs`, and `docker compose down -v`.

---

## Safe Customizations & Configuration Reference

| Target File | Permitted Customizations | Important Cautions |
| :--- | :--- | :--- |
| `app.py` | Change HTML heading, table styles, CSS colors | Do **NOT** change `host="db"` or environment variable names |
| `init.sql` | Add more sample rows, change student names/courses | Must be edited before first run, or run `docker compose down -v` to re-initialize |
| `docker-compose.yml` | Change passwords, database names, or host port (e.g. `8080:80`) | If changed, update both `web` and `db` environment blocks and the `docker compose exec` command |

---

## Troubleshooting Guide

| Issue / Error | Root Cause | Solution |
| :--- | :--- | :--- |
| `mapping values are not allowed here` | Incorrect YAML indentation or tabs used | Re-check `docker-compose.yml`. YAML must use 2 spaces per indentation level, never tabs |
| `empty compose file` or file size `0` bytes | File not saved properly in editor | Open file in Notepad or VS Code, paste content, save with `Ctrl + S`, and verify with `dir` |
| File saved as `app.py.txt` | Notepad automatically appended `.txt` extension | In PowerShell: `Rename-Item app.py.txt app.py` |
| `Database connection failed: 2003` | MySQL container still initializing | MySQL 8.0 takes ~15-25 seconds to start. Wait for `docker compose ps` to show `(healthy)` and refresh |
| `Table 'studentdb.students' doesn't exist` | An existing volume skipped running `init.sql` | Teardown existing volume: `docker compose down -v`, then rerun `docker compose up -d --build` |
| `port is already allocated: 8000` | Another process is listening on port 8000 | Find and terminate conflicting process or container: `docker ps` and `docker stop <id>` |

---

## Screenshots Checklist

- [ ] **1.** `docker compose ps` showing `compose-app-db-1` as `healthy` and `compose-app-web-1` as `Up`
- [ ] **2.** Browser view at `http://localhost:8000` showing student records table
- [ ] **3.** Direct MySQL query output (`docker compose exec db ...`)
- [ ] **4.** Scaling demonstration (`--scale web=0` and `--scale web=1`)
- [ ] **5.** Logs output (`docker compose logs web` and `docker compose logs db`)
- [ ] **6.** Teardown with volume removal (`docker compose down -v` and `docker volume ls`)
