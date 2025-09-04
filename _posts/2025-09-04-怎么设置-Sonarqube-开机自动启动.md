---
layout: post
title: "怎么设置Sonarqube开机自动启动"
date: 2025-09-04
categories: sonarqube
tags: [sonarqube, service, script]
---

# 怎么用脚本设置Sonarqube开机自动启动


通过一个 **一键运行的脚本**，执行后就能自动创建 systemd 服务，并设置好开机自启。

假设：

* SonarQube 安装在 `/opt/sonarqube`
* SonarQube 启动脚本路径：`/opt/sonarqube/bin/linux-x86-64/sonar.sh`
* 运行用户：`sonarqube`

创建一个脚本 `setup-sonarqube-service.sh`，执行后就能完成：

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

echo "设置开机自启..."
systemctl enable sonarqube

echo "完成！使用以下命令管理 SonarQube:"
echo "  systemctl start sonarqube"
echo "  systemctl stop sonarqube"
echo "  systemctl restart sonarqube"
echo "  systemctl status sonarqube"
```

---

### 使用方法

```bash
sudo bash setup-sonarqube-service.sh
```

执行后，SonarQube 就会被注册为 systemd 服务，并且设置开机自启。

---

全文结束 

2025-09-04
