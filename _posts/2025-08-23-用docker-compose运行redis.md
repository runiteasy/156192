
# 怎么用 Docker Compose 运行 Redis

---

## 准备工作

### 环境要求

本文的步骤主要基于 **Linux** 系统，如果你使用 **Windows** 或 **macOS** 上的 Docker Desktop 同样可以完成。

**实验环境：**

* Ubuntu 24.04.3 LTS
* Docker version 28.3.3, build 980b856
* Docker Compose version v2.39.1

### 安装 Docker

如果你还没有安装 Docker，可以参考官方文档进行安装：https://docs.docker.com/engine/install/

新版 Docker Engine 默认已包含 Docker Compose，无需单独安装。

**注意：** 如果你的网络环境无法连接到 Docker Hub，可以配置代理服务器或使用国内镜像仓库。

#### 配置代理服务器

修改 `/etc/docker/daemon.json` 文件，添加以下内容来为 Docker 守护进程配置代理。

**注意：** 将 `http://192.168.57.1:808` 替换为你的代理服务器地址和端口。`"no-proxy"` 字段的作用是排除本地地址不走代理。

```json
{
    "proxies": {
        "http-proxy": "http://192.168.57.1:808",
        "https-proxy": "http://192.168.57.1:808",
        "no-proxy": "localhost,127.0.0.1,::1"
    }
}
````

修改后，执行以下命令重启 Docker 服务：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### 使用国内镜像仓库

为了加速镜像下载，你也可以配置国内镜像仓库。在修改 `/etc/docker/daemon.json` 文件时添加如下内容，将 `https://docker.xuanyuan.me` 替换为可用的镜像仓库地址。

```json
{
    "registry-mirrors": [
        "https://docker.xuanyuan.me"
    ]
}
```

你可以通过搜索引擎或询问 AI 找到可用的镜像仓库。这里提供一个知乎上的镜像列表供参考：https://zhuanlan.zhihu.com/p/24461370776

修改后，执行以下命令重启 Docker 服务：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

-----

## 第一步：创建配置文件

首先，创建一个独立的文件夹用于存放 Redis 的配置文件，然后进入该文件夹。

```bash
mkdir ./redis
cd ./redis
```

接下来，创建 `docker-compose.yml` 文件。

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

**文件解释：**

  * `image: redis:latest`：指定使用最新版的 Redis 镜像。
  * `container_name: docker_redis`：为容器指定一个易于识别的名称。
  * `ports: - "6379:6379"`：将宿主机的 6379 端口映射到容器的 6379 端口，这样可以从外部访问 Redis。
  * `command`：在启动容器时执行的命令，这里指定了持久化模式和 ACL 权限文件。
  * `volumes`：将宿主机的目录或文件挂载到容器内部，实现数据持久化和配置共享。

然后，创建 `users.acl` 文件来配置 Redis 的用户权限。

```bash
touch users.acl

cat << EOF > users.acl
user default off -@all
user redisuser on >redispassword +@all ~*
EOF
```

**文件解释：**

  * `user default off -@all`：禁用默认用户，提升安全性。
  * `user redisuser on >redispassword +@all ~*`：创建一个名为 `redisuser` 的新用户，并设置密码为 `redispassword`，同时赋予所有命令的执行权限。

-----

## 第二步：启动 Redis 容器

在 **redis** 文件夹中，执行以下命令启动容器。`docker compose` 会自动从 Docker Hub 拉取 `redis:latest` 镜像。

`-d` 参数表示在后台（detached mode）运行容器。

```bash
docker compose up -d
```

稍等片刻，执行 `docker compose ps` 命令检查容器状态。如果返回类似下面的输出，则说明启动成功。

```bash
NAME           IMAGE          COMMAND                  SERVICE   CREATED       STATUS       PORTS
docker_redis   redis:latest   "docker-entrypoint.s…"   redis     2 hours ago   Up 2 hours   0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp
```

-----

## 第三步：验证 Redis 是否正常运行

你可以选择以下两种方式来测试 Redis 是否正常工作。

### 方式一：使用 `redis-cli` 命令行工具

首先，安装 `redis-cli`。

```bash
sudo apt-get update
sudo apt-get install redis-tools
```

然后，使用 `redis-cli` 连接到 Redis 实例。`-h` 用于指定主机地址，`-p` 用于指定端口。

```bash
redis-cli -h 127.0.0.1 -p 6379
```

连接后，使用 `auth` 命令进行身份验证。

```bash
127.0.0.1:6379> auth redisuser redispassword
OK
127.0.0.1:6379> set mytestkey 1
OK
127.0.0.1:6379> get mytestkey
"1"
127.0.0.1:6379> exit
```

### 方式二：进入容器内部测试

使用 `docker exec` 命令进入正在运行的容器。

```bash
docker exec -it docker_redis redis-cli
```

在容器内执行 `redis-cli` 后，同样进行身份验证和测试。

```bash
127.0.0.1:6379> auth redisuser redispassword
OK
127.0.0.1:6379> set mytestkey 1
OK
127.0.0.1:6379> get mytestkey
"1"
127.0.0.1:6379> exit
```

如果两种方式都返回相同的结果，恭喜你，Redis 已经成功运行！现在你可以在自己的程序中连接和使用 Redis 了。

-----

本文结束

2025年08月23日

