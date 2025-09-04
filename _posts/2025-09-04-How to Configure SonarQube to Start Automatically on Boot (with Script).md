---
layout: post
title: "How to Configure SonarQube to Start Automatically on Boot (with Script)"
date: 2025-09-04
categories: sonarqube
tags: [sonarqube, service, script, systemd, linux, ubuntu, rhel]
description: "Learn how to configure SonarQube to start automatically on system boot using a simple shell script and systemd service configuration. Works for Ubuntu, RHEL, and CentOS."
keywords: "SonarQube auto start, SonarQube service, SonarQube systemd, SonarQube startup script, SonarQube Ubuntu, SonarQube RHEL, SonarQube CentOS"
---

# How to Configure SonarQube to Start Automatically on Boot Using a Script

If you have installed **SonarQube** on Linux (Ubuntu, RHEL, CentOS, etc.), you might notice that it doesn’t start automatically after a system reboot.  
In this guide, I’ll show you how to create a **one-click script** that registers SonarQube as a `systemd` service and enables it to start automatically on boot.

---

## Assumptions

- SonarQube is installed in: `/opt/sonarqube`  
- SonarQube startup script: `/opt/sonarqube/bin/linux-x86-64/sonar.sh`  
- Running user: `sonarqube`  

---

## Step 1: Create the script

Create a script named `setup-sonarqube-service.sh` with the following content:

```bash
#!/bin/bash
# setup-sonarqube-service.sh
# Configure SonarQube systemd service

SONAR_USER="sonarqube"
SONAR_DIR="/opt/sonarqube"
SONAR_BIN="$SONAR_DIR/bin/linux-x86-64/sonar.sh"
SERVICE_FILE="/etc/systemd/system/sonarqube.service"

# Check if the user exists
if ! id "$SONAR_USER" &>/dev/null; then
  echo "Creating SonarQube user: $SONAR_USER"
  useradd -r -s /bin/bash "$SONAR_USER"
  chown -R $SONAR_USER:$SONAR_USER $SONAR_DIR
fi

# Create systemd service file
echo "Creating systemd service file: $SERVICE_FILE"
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

# Reload systemd
echo "Reloading systemd configuration..."
systemctl daemon-reload

# Start service and enable auto start
echo "Starting SonarQube service..."
systemctl start sonarqube

echo "Enabling SonarQube auto start..."
systemctl enable sonarqube

echo "Done! Use the following commands to manage SonarQube:"
echo "  systemctl start sonarqube"
echo "  systemctl stop sonarqube"
echo "  systemctl restart sonarqube"
echo "  systemctl status sonarqube"
````

---

## Step 2: Run the script

```bash
sudo bash setup-sonarqube-service.sh
```

Once executed, SonarQube will be registered as a systemd service and automatically start on boot.

---

## Conclusion

With this script, you don’t need to manually start SonarQube after every reboot.
This method works on **Ubuntu, Debian, RHEL, CentOS, and most systemd-based Linux distributions**.

---

*Published on 2025-09-04*
