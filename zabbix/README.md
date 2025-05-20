# Zabbix Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Zabbix is a powerful open-source monitoring solution for networks, servers, cloud services, and applications. This guide provides step-by-step instructions to install and configure the Zabbix server, frontend, and agent on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                                              | Recommended |
| -------- | -------------------------------------------------------------------- | ----------- |
| CPU      | 2 cores                                                              | 4 cores     |
| RAM      | 2 GB                                                                 | 4 GB+       |
| Disk     | 10 GB                                                                | 40 GB+      |
| Network  | Required for communication between server, agents, and web interface |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Database: MySQL/MariaDB
- Web Server: Nginx or Apache
- PHP: PHP 8.1+
- SELinux: Permissive or properly configured
- Required Packages: mariadb-server, httpd, php, zabbix-server-mysql, zabbix-web-mysql, zabbix-agent

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Install Zabbix Repository

```bash
sudo dnf install -y https://repo.zabbix.com/zabbix/6.4/rhel/9/x86_64/zabbix-release-6.4-1.el9.noarch.rpm
sudo dnf clean all
```

### 3.3 Install Zabbix Server, Web Frontend, Agent

```bash
sudo dnf install -y zabbix-server-mysql zabbix-web-mysql zabbix-agent
```

---

## 4. Configure Database

### 4.1 Install and Start MariaDB

```bash
sudo dnf install -y mariadb-server
sudo systemctl enable --now mariadb
```

### 4.2 Create Zabbix Database and User

```bash
mysql -uroot
```
```bash
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER zabbix@localhost IDENTIFIED BY 'zabbix_password';
GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost;
FLUSH PRIVILEGES;
EXIT;
```

### 4.3 Import Initial Schema

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
```

---

## 5. Configure Zabbix Server

Edit configuration file:
```bash
sudo vim /etc/zabbix/zabbix_server.conf
```
Set:
```bash
DBPassword=zabbix_password
```

---

## 6. Configure PHP for Zabbix Frontend

Edit Zabbix web config:
```bash
sudo vim /etc/php-fpm.d/zabbix.conf
```
Ensure:
```bash
php_value[date.timezone] = Asia/Shanghai
```

---

## 7. Start Services

```bash
sudo systemctl enable --now zabbix-server zabbix-agent php-fpm httpd
```
Check status:
```bash
sudo systemctl status zabbix-server
```

---

## 8. Configure Firewall

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=10051/tcp
sudo firewall-cmd --permanent --add-port=10050/tcp
sudo firewall-cmd --reload
```

---

## 9. Initial Setup

### 9.1 Access Web Interface

Open a browser:
```bash
http://<your-server-ip>/zabbix
```

### 9.2 Complete Web Setup

- Choose language and timezone
- Check prerequisites
- Enter database details
- Configure Zabbix server name and connection
- Finish installation

### 9.3 Login

Default credentials:
```bash
Username: Admin
Password: zabbix
```

---

## 10. Post-Installation Configuration

### 10.1 Change Admin Password

After first login, change the default password.

### 10.2 Add Hosts

Go to:
```bash
Configuration → Hosts → Create host
```
Configure host name, interfaces, templates.

### 10.3 Set Email Notifications (Optional)

Go to:
```bash
Administration → Media types → Email
```
Set SMTP server and user details.

---

## 11. Backup & Restore

### 11.1 Backup

Stop Zabbix:
```bash
sudo systemctl stop zabbix-server
```
Backup configuration and database:
```bash
sudo tar -czvf zabbix-config-backup.tar.gz /etc/zabbix
mysqldump -uzabbix -p zabbix > zabbix-db.sql
```
Start Zabbix:
```bash
sudo systemctl start zabbix-server
```

### 11.2 Restore

Stop services and restore:
```bash
sudo systemctl stop zabbix-server
sudo tar -xzvf zabbix-config-backup.tar.gz -C /
mysql -uzabbix -p zabbix < zabbix-db.sql
sudo systemctl start zabbix-server
```

---

## 12. Logging & Monitoring

- Server log: /var/log/zabbix/zabbix_server.log
- Agent log: /var/log/zabbix/zabbix_agentd.log
- Web server logs: /var/log/httpd/
Monitor services:
```bash
sudo systemctl status zabbix-server
sudo systemctl status zabbix-agent
```

---

## 13. Troubleshooting

| Issue                      | Resolution                                |
| -------------------------- | ----------------------------------------- |
| Web UI not loading         | Check httpd and php-fpm status            |
| Database errors            | Check DB credentials and connectivity     |
| Zabbix server not starting | Inspect zabbix\_server.log                |
| No data from agent         | Verify agent is running and reachable     |
| Wrong time displayed       | Check timezone settings in PHP and system |

---

## 14. Uninstallation (if needed)

```bash
sudo systemctl stop zabbix-server zabbix-agent httpd mariadb
sudo dnf remove zabbix-server-mysql zabbix-agent zabbix-web-mysql httpd mariadb-server
sudo rm -rf /etc/zabbix /var/lib/mysql /var/log/zabbix
```

---

## 15. Appendix

### 15.1 Useful Commands

```bash
sudo systemctl start zabbix-server
sudo systemctl stop zabbix-server
sudo systemctl restart zabbix-server
sudo systemctl status zabbix-server
```

### 15.2 Important Paths

| Description  | Path                               |
| ------------ | ---------------------------------- |
| Config files | /etc/zabbix                        |
| Server log   | /var/log/zabbix/zabbix\_server.log |
| Agent log    | /var/log/zabbix/zabbix\_agentd.log |
| Web UI path  | /usr/share/zabbix                  |

### 15.3 References

- Zabbix: https://www.zabbix.com
- Docs: https://www.zabbix.com/documentation/

---

## 16. Document History

| Version | Date       | Changes                    | Author   |
| ------- | ---------- | -------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for Zabbix | xianzhan |
