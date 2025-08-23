# How to Run Redis with Docker Compose

---

## Prerequisites

### Environment Requirements

The steps in this guide are mainly based on **Linux** systems. If you are using **Windows** or **macOS** with Docker Desktop, you can follow along as well.

**Test Environment:**

* Ubuntu 24.04.3 LTS
* Docker version 28.3.3, build 980b856
* Docker Compose version v2.39.1

### Install Docker

If you have not installed Docker yet, refer to the official documentation: https://docs.docker.com/engine/install/

The latest Docker Engine already includes Docker Compose, so no need for separate installation.

---

## Step 1: Create Configuration Files

First, create a folder to store Redis configuration files and navigate into it:

```bash
mkdir ./redis
cd ./redis
```

Next, create the `docker-compose.yml` file:

```bash
cat << EOF > docker-compose.yml
services:
  redis:
    image: redis:latest
    container_name: docker_redis
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes --aclfile /etc/redis/users.acl
    volumes:
      - ./data/redis:/data
      - ./users.acl:/etc/redis/users.acl
EOF
```

**Explanation:**

  * `image: redis:latest`: Use the latest Redis image.
  * `container_name: docker_redis`: Name the container for easier identification.
  * `ports: - "6379:6379"`: Map host port 6379 to container port 6379.
  * `command`: Command executed at container start, enabling persistence and ACL file.
  * `volumes`: Mount host directories/files for persistence and configuration sharing.

Now, create the `users.acl` file to configure Redis users:

```bash
touch users.acl

cat << EOF > users.acl
user default off -@all
user redisuser on >redispassword +@all ~*
EOF
```

**Explanation:**

  * `user default off -@all`: Disable the default user for security.
  * `user redisuser on >redispassword +@all ~*`: Create `redisuser` with password `redispassword` and grant full permissions.

---

## Step 2: Start Redis Container

From the **redis** folder, start the container with:

```bash
docker compose up -d
```

The `-d` flag runs the container in detached mode. Docker Compose will pull the `redis:latest` image from Docker Hub.

Check container status:

```bash
docker compose ps
```

Expected output if successful:

```bash
NAME           IMAGE          COMMAND                  SERVICE   CREATED       STATUS       PORTS
docker_redis   redis:latest   "docker-entrypoint.s…"   redis     2 hours ago   Up 2 hours   0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp
```

---

## Step 3: Verify Redis is Running

You can test Redis in two ways.

### Method 1: Use `redis-cli`

Install the CLI tool:

```bash
sudo apt-get update
sudo apt-get install redis-tools
```

Connect to Redis:

```bash
redis-cli -h 127.0.0.1 -p 6379
```

Authenticate and test:

```bash
127.0.0.1:6379> auth redisuser redispassword
OK
127.0.0.1:6379> set mytestkey 1
OK
127.0.0.1:6379> get mytestkey
"1"
127.0.0.1:6379> exit
```

### Method 2: Inside the Container

Exec into the running container:

```bash
docker exec -it docker_redis redis-cli
```

Authenticate and test inside the container:

```bash
127.0.0.1:6379> auth redisuser redispassword
OK
127.0.0.1:6379> set mytestkey 1
OK
127.0.0.1:6379> get mytestkey
"1"
127.0.0.1:6379> exit
```

If both methods return the same results, Redis is running successfully. You can now connect to Redis from your applications.

---

End of Article

August 23, 2025
