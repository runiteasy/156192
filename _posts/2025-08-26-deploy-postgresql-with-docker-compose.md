How to Deploy Postgresql with Docker Compose
---
# **Deploying Bitnami PostgreSQL on Ubuntu Using Docker Compose (Persistent Data + Permission Fix)**

> This guide explains how to set up a **Bitnami PostgreSQL** container using **Docker Compose** on Ubuntu, configure persistent storage, and fix permission issues such as
> `FATAL: could not open file "global/pg_filenode.map": Permission denied`.

---

## **1. Create the Directory Structure**

On your Ubuntu host, create a directory to store PostgreSQL data and configuration:

```bash
mkdir -p ~/bitnami-postgres/postgresql_data
chown 1001:1001 ~/bitnami-postgres/postgresql_data
```

> **Explanation**
>
> * `postgresql_data`: Directory for PostgreSQL data (persistent storage).
> * UID `1001`: The default user ID used by Bitnami PostgreSQL inside the container.

---

## **2. Create the `docker-compose.yml` File**

Create a `docker-compose.yml` file inside `~/bitnami-postgres`:

```yaml
version: '3.8'

services:
  docker_postgres:
    image: bitnami/postgresql:latest
    container_name: docker_postgres
    restart: always
    environment:
      - POSTGRESQL_USERNAME=dockeruser
      - POSTGRESQL_PASSWORD=yourpassword
      - POSTGRESQL_DATABASE=dockerdatabase
    ports:
      - "5432:5432"
    volumes:
      - ./postgresql_data:/bitnami/postgresql
```

> **Note**
> Bitnami PostgreSQL stores data in `/bitnami/postgresql` inside the container,
> unlike the official PostgreSQL image, which uses `/var/lib/postgresql/data`.

---

## **3. Start PostgreSQL**

Start the container:

```bash
docker compose up -d
```

Check logs:

```bash
docker compose logs docker_postgres
```

If PostgreSQL starts successfully, you should see:

```
postgresql  INFO  ==> Starting PostgreSQL in background...
postgresql  INFO  ==> PostgreSQL started successfully
```

---

## **4. Common Issues & Fixes**

### **Issue: pg\_filenode.map Permission Denied**

If you see errors in the logs or `psql` outputs:

```
FATAL: could not open file "global/pg_filenode.map": Permission denied
```

This means **your mounted data directory permissions are incorrect**.

---

### **4.1 Check UID/GID Inside the Container**

Run:

```bash
docker exec -it docker_postgres ls -ld /bitnami/postgresql/data
```

Example output:

```
drwx------ 19 1001 1001 4096 Aug 27 10:46 data
```

If **UID=1001**, PostgreSQL runs under UID `1001`.
If your host-mounted directory isn’t owned by UID `1001`, PostgreSQL cannot access the data.

---

### **4.2 Fix Permissions on the Host**

Run the following on the host machine:

```bash
sudo chown -R 1001:1001 ~/bitnami-postgres/postgresql_data
```

Verify:

```bash
ls -ld ~/bitnami-postgres/postgresql_data
```

Expected output:

```
drwx------ 19 1001 1001 4096 Aug 27 10:46 postgresql_data
```

---

### **4.3 Restart the Container**

```bash
docker compose down
docker compose up -d
```

Check logs again:

```bash
docker compose logs docker_postgres
```

If no errors appear, the permission issue is fixed.

---

## **5. Test if PostgreSQL Is Working**

### **5.1 Inside the Container**

```bash
docker exec -it docker_postgres psql -U dockeruser -d dockerdatabase -c "SELECT 1;"
```

Expected result:

```
 ?column?
----------
        1
(1 row)
```

---

### **5.2 From the Host**

If port `5432` is mapped, test from the host:

```bash
psql -h 127.0.0.1 -p 5432 -U dockeruser -d dockerdatabase -c "SELECT 1;"
```

---

## **6. Summary**

| Feature              | Directory / Config                         | Description                             |
| -------------------- | ------------------------------------------ | --------------------------------------- |
| **Data Directory**   | `~/bitnami-postgres/postgresql_data`       | **Must be persistent**; ensure UID=1001 |
| **Config Directory** | `~/bitnami-postgres/config` *(optional)*   | Custom `postgresql.conf` if needed      |
| **Image**            | `bitnami/postgresql`                       | Bitnami PostgreSQL container            |
| **Default UID**      | `1001`                                     | Change ownership if permission issues   |
| **Start Command**    | `docker compose up -d`                     | Start container in background           |
| **Test Command**     | `docker exec -it docker_postgres psql ...` | Verify database connectivity            |

---

## **7. Final Result**

* Data is **persistent** ✅
* Permissions are **correct** ✅
* PostgreSQL **runs smoothly** ✅
* Database **connections work** ✅

---


End of Article

August 26, 2025
