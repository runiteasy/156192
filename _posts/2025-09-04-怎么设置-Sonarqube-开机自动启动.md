---
layout: post
title: "SonarQube 开机自动启动配置（脚本方式）"
date: 2025-09-04
categories: sonarqube
tags: [sonarqube, service, script, systemd, linux, ubuntu, rhel]
description: "通过一个简单的 shell 脚本，在 Linux 上配置 SonarQube 为 systemd 服务，实现开机自动启动。适用于 Ubuntu、RHEL、CentOS 等系统。"
keywords: "SonarQube 开机自启, SonarQube 自动启动, SonarQube systemd, SonarQube 服务, SonarQube 脚本, SonarQube Ubuntu, SonarQube RHEL, SonarQube CentOS"
---

# SonarQube 开机自动启动配置（脚本方式）

在 Linux 系统（Ubuntu、RHEL、CentOS 等）中安装 **SonarQube** 后，默认情况下系统重启后不会自动启动。  
本文将演示如何通过一个 **一键脚本** 把 SonarQube 注册为 `systemd` 服务，并设置开机自启。

---

## 环境假设

- SonarQube 安装路径：`/opt/sonarqube`  
- SonarQube 启动脚本：`/opt/sonarqube/bin/linux-x86-64/sonar.sh`  
- 运行用户：`sonarqube`  

---

## 步骤一：创建脚本

新建一个脚本 `setup-sonarqube-service.sh`，内容如下：

```bash
#!/bin/bash
# setup-sonarqube-service.sh
# 配置 SonarQube systemd 服务

SONAR_USER="sonarqube"
SONAR_DIR="/opt/sonarqube"
SONAR_BIN="$SONAR_DIR/bin/linux-x86-64/sonar.sh"
SERVICE_FILE="/etc/systemd/system/sonarqube.service"

# 检查是否存在运行用户
if ! id "$SONAR_USER" &>/dev/null; then
  echo "创建 SonarQube 用户: $SONAR_USER"
  useradd -r -s /bin/bash "$SONAR_USER"
  chown -R $SONAR_USER:$SONAR_USER $SONAR_DIR
fi

# 创建 systemd 服务文件
echo "创建 systemd 服务文件: $SERVICE_FILE"
cat > $SERVICE_FILE <<EOF
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=$SONAR_BIN start
ExecStop=$SONAR_BIN stop
ExecReload=$SONAR_BIN restart
User=$SONAR_USER
Group=$SONAR_USER
Restart=always
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 重新加载 systemd 配置
echo "重新加载 systemd 配置..."
systemctl daemon-reload

# 启动服务并设置开机自启
echo "启动 SonarQube 服务..."
systemctl start sonarqube

echo "设置 SonarQube 开机自启..."
systemctl enable sonarqube

echo "完成！使用以下命令管理 SonarQube:"
echo "  systemctl start sonarqube"
echo "  systemctl stop sonarqube"
echo "  systemctl restart sonarqube"
echo "  systemctl status sonarqube"
````

---

## 步骤二：运行脚本

```bash
sudo bash setup-sonarqube-service.sh
```

执行完成后，SonarQube 将被注册为 systemd 服务，并自动设置为开机启动。

---

## 总结

通过这种脚本方式，可以让 SonarQube 在系统重启后自动运行，避免每次手动启动。
该方法适用于 **Ubuntu、Debian、RHEL、CentOS 以及其他基于 systemd 的 Linux 发行版**。

---

*发布日期：2025-09-04*

