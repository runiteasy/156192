
# **Ubuntu 上使用 Docker Compose 部署 Bitnami PostgreSQL（持久化 + 权限修复）**

> 本文档适用于 **Bitnami PostgreSQL** 容器（镜像：`bitnami/postgresql`）。

---

## **1. 创建目录结构**

在 Ubuntu 上创建用于存储数据和配置的目录：

```bash
mkdir -p ~/bitnami-postgres/postgresql_data
chown 1001:1001 ~/bitnami-postgres/postgresql_data
```

> **说明**
>
> * `postgresql_data`：数据库数据目录（持久化）


---

## **2. 编写 docker-compose.yml**

在 `~/bitnami-postgres` 目录下创建 `docker-compose.yml`：

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

> **注意**
> Bitnami 的 PostgreSQL 镜像会在容器内使用 `/bitnami/postgresql` 作为数据目录，而不是官方镜像的 `/var/lib/postgresql/data`。

---

## **3. 启动 PostgreSQL**

```bash
docker compose up -d
```

查看日志：

```bash
docker compose logs docker_postgres
```

如果 PostgreSQL 启动正常，你会看到类似：

```
postgresql  INFO  ==> Starting PostgreSQL in background...
postgresql  INFO  ==> PostgreSQL started successfully
```

---

## **4. 常见错误与解决方法**

### **错误：pg\_filenode.map 权限被拒绝**

如果日志或 `psql` 提示：

```
FATAL: could not open file "global/pg_filenode.map": Permission denied
```

这是因为**挂载的数据目录权限不对**。

---

### **4.1 确认容器内的数据目录 UID/GID**

进入容器查看：

```bash
docker exec -it docker_postgres ls -ld /bitnami/postgresql/data
```

示例输出：

```
drwx------ 19 1001 1001 4096 Aug 27 10:46 data
```

如果 **UID=1001**，则 PostgreSQL 进程使用 UID=1001 运行。
但如果你的宿主机数据目录属主不是 1001，就会报 `Permission denied`。

---

### **4.2 修复宿主机目录权限**

在宿主机上执行：

```bash
sudo chown -R 1001:1001 ~/bitnami-postgres/postgresql_data
```

重新检查：

```bash
ls -ld ~/bitnami-postgres/postgresql_data
```

应该显示：

```
drwx------ 19 1001 1001 4096 Aug 27 10:46 postgresql_data
```

---

### **4.3 重启容器**

```bash
docker compose down
docker compose up -d
```

再次查看日志：

```bash
docker compose logs docker_postgres
```

如果没有报错，说明权限修复成功。

---

## **5. 测试 PostgreSQL 是否可用**

### **5.1 容器内测试**

```bash
docker exec -it docker_postgres psql -U dockeruser -d dockerdatabase -c "SELECT 1;"
```

如果 PostgreSQL 正常工作，会返回：

```
 ?column?
----------
        1
(1 row)
```

---

### **5.2 宿主机测试**

如果你映射了 5432 端口，可以直接在宿主机测试：

```bash
psql -h 127.0.0.1 -p 5432 -U dockeruser -d dockerdatabase -c "SELECT 1;"
```

---

## **6. 总结**

| 功能     | 目录 / 配置                                 | 说明                             |
| ------ | --------------------------------------- | ------------------------------ |
| 数据目录   | `~/bitnami-postgres/postgresql_data`    | **必须持久化**，确保 UID=1001          |
| 配置目录   | `~/bitnami-postgres/config`             | 可选，存放 `postgresql.conf`        |
| 镜像     | `bitnami/postgresql`                    | Bitnami 版 PostgreSQL           |
| 默认 UID | `1001`                                  | 如果权限错误，需要 `chown -R 1001:1001` |
| 启动命令   | `docker compose up -d`                  | 后台启动                           |
| 测试命令   | `docker exec -it docker_postgres psql ...` | 检查数据库是否正常                      |

---

## **最终效果**

* 数据持久化 ✅
* 权限正确 ✅
* PostgreSQL 正常运行 ✅
* 可随时连接测试 ✅

---

全文结束

2025年08月27日