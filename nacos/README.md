# Nacos Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Nacos (Naming and Configuration Service) is an easy-to-use platform designed for dynamic service discovery, configuration management, and service management. This guide provides step-by-step instructions to install and configure Nacos on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                       | Recommended |
| -------- | --------------------------------------------- | ----------- |
| CPU      | 2 cores                                       | 4 cores     |
| RAM      | 2 GB                                          | 4 GB+       |
| Disk     | 5 GB                                          | 10 GB+      |
| Network  | Required for cluster and client communication |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Java: OpenJDK 8 or 11
- Database: MySQL 5.7 or 8.0 (for production mode)
- SELinux: Permissive or properly configured
- Dependencies: wget, unzip, firewalld, systemd, curl

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Install Java

```bash
sudo dnf install -y java-11-openjdk java-11-openjdk-devel
```
Verify installation:
```bash
java -version
```

### 3.3 Create Nacos User and Directory

```bash
sudo useradd -M -s /sbin/nologin nacos
sudo mkdir -p /opt/nacos
sudo chown -R nacos:nacos /opt/nacos
```

### 3.4 Download and Extract Nacos

```bash
cd /opt
sudo wget https://github.com/alibaba/nacos/releases/download/2.3.2/nacos-server-2.3.2.tar.gz
sudo tar -xzf nacos-server-2.3.2.tar.gz -C /opt/nacos --strip-components=1
sudo chown -R nacos:nacos /opt/nacos
```

---

## 4. Configure Firewall

Default port for Nacos is 8848:
```bash
sudo firewall-cmd --permanent --add-port=8848/tcp
sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Configure Database (Optional for Production)

Create nacos_config database in MySQL and import the SQL script:
```bash
mysql -u root -p < /opt/nacos/conf/nacos-mysql.sql
```
Update application.properties:
```bash
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=your_password
```

### 5.2 Start Nacos Server

```bash
sudo su - nacos
cd /opt/nacos/bin
./startup.sh -m standalone
```
For cluster mode, use -m cluster and configure cluster.conf.

---

## 6. Post-Installation Configuration

### 6.1 Access Nacos Dashboard

Open in browser:
```bash
http://<your-server-ip>:8848/nacos
```
Default credentials:
- Username: nacos
- Password: nacos

### 6.2 Optional: Systemd Service

Create service file:
```bash
sudo vi /etc/systemd/system/nacos.service
```
Paste:
```bash
[Unit]
Description=Nacos Server
After=network.target

[Service]
User=nacos
ExecStart=/opt/nacos/bin/startup.sh -m standalone
ExecStop=/opt/nacos/bin/shutdown.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Enable and start:
```bash
sudo systemctl daemon-reexec
sudo systemctl enable nacos
sudo systemctl start nacos
```

---

## 7. Backup & Restore

### 7.1 Backup

Stop Nacos:
```bash
sudo systemctl stop nacos
```
Backup:
```bash
sudo tar -czvf nacos-backup.tar.gz /opt/nacos
```
Start Nacos:
```bash
sudo systemctl start nacos
```

### 7.2 Restore

Stop service:
```bash
sudo systemctl stop nacos
```
Restore:
```bash
sudo tar -xzvf nacos-backup.tar.gz -C /
```
Ensure permissions:
```bash
sudo chown -R nacos:nacos /opt/nacos
```
Start service:
```bash
sudo systemctl start nacos
```

---

## 8. Logging & Monitoring

- Log files: /opt/nacos/logs/
- View logs:
```bash
tail -f /opt/nacos/logs/start.out
tail -f /opt/nacos/logs/nacos.log
```
- Optional: Use Prometheus + Grafana or Nacos monitoring plugins

---

## 9. Troubleshooting

| Issue                    | Resolution                                        |
| ------------------------ | ------------------------------------------------- |
| Nacos not starting       | Check logs in `/opt/nacos/logs/start.out`         |
| Port 8848 not accessible | Check firewalld and Nacos status                  |
| DB connection error      | Verify MySQL settings in `application.properties` |
| Slow dashboard           | Increase RAM, check JVM settings                  |
| Web UI login fails       | Reset password in DB or check login config        |

---

## 10. Uninstallation (if needed)

```bash
sudo systemctl stop nacos
sudo systemctl disable nacos
sudo rm -rf /opt/nacos /etc/systemd/system/nacos.service
sudo userdel nacos
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
./startup.sh -m standalone
./startup.sh -m cluster
./shutdown.sh
```

### 11.2 Important Paths

| Description | Path            |
| ----------- | --------------- |
| Nacos home  | /opt/nacos      |
| Config file | /opt/nacos/conf |
| Log files   | /opt/nacos/logs |

### 11.3 References

- Nacos: https://nacos.io
- Docs: https://nacos.io/en-us/docs/
- GitHub: https://github.com/alibaba/nacos

---

## 12. Document History

| Version | Date       | Changes                   | Author   |
| ------- | ---------- | ------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for Nacos | xianzhan |

---
