# MySQL Deployment Guide (Rocky Linux 9)

---

## 1. Overview

MySQL is a popular open-source relational database management system (RDBMS) used widely for web applications and enterprise systems. This guide provides step-by-step instructions to install and configure MySQL on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                             | Recommended |
| -------- | ----------------------------------- | ----------- |
| CPU      | 1 core                              | 2 cores     |
| RAM      | 1 GB                                | 2 GB+       |
| Disk     | 5 GB                                | 10 GB+      |
| Network  | Required for remote database access |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- SELinux: Permissive or configured
- Packages: wget, curl, systemd, firewalld (optional for remote access)

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Add MySQL Repository

```bash
sudo dnf install -y https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf repolist enabled | grep mysql
```

### 3.3 Install MySQL Server

```bash
sudo dnf install -y mysql-community-server
```

---

## 4. Configure Firewall (Optional for Remote Access)

```bash
sudo firewall-cmd --permanent --add-service=mysql
sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Enable and Start MySQL

```bash
sudo systemctl enable mysqld
sudo systemctl start mysqld
```

### 5.2 Get Temporary Root Password

```bash
sudo grep 'temporary password' /var/log/mysqld.log
```

### 5.3 Secure MySQL Installation

Run the following command and follow prompts to set root password, remove anonymous users, disallow root remote login, etc.:
```bash
sudo mysql_secure_installation
```

---

## 6. Post-Installation Configuration

### 6.1 Connect to MySQL

```bash
mysql -u root -p
```

### 6.2 Create a Database and User

```bash
CREATE DATABASE example_db;
CREATE USER 'example_user'@'%' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON example_db.* TO 'example_user'@'%';
FLUSH PRIVILEGES;
```

### 6.3 Allow Remote Connections (Optional)

Edit MySQL config:
```bash
sudo vi /etc/my.cnf
```
Under [mysqld], add:
```bash
bind-address=0.0.0.0
```
Restart service:
```bash
sudo systemctl restart mysqld
```

---

## 7. Backup & Restore

### 7.1 Backup

```bash
mysqldump -u root -p example_db > example_db_backup.sql
```

### 7.2 Restore

```bash
mysql -u root -p example_db < example_db_backup.sql
```

---

## 8. Logging & Monitoring

- Log file: /var/log/mysqld.log
- Check service status:
```bash
sudo systemctl status mysqld
```

---

## 9. Troubleshooting

| Issue                  | Resolution                                        |
| ---------------------- | ------------------------------------------------- |
| Can't connect to MySQL | Check service, firewall, bind-address             |
| Access denied          | Ensure correct user/password and host permissions |
| Slow queries           | Enable slow query log and optimize queries        |
| Temp password expired  | Reset via `mysqladmin` or SQL                     |

---

## 10. Uninstallation (if needed)

```bash
sudo systemctl stop mysqld
sudo dnf remove -y mysql-community-server
sudo rm -rf /var/lib/mysql /etc/my.cnf /var/log/mysqld.log
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
sudo systemctl start mysqld
sudo systemctl stop mysqld
sudo systemctl restart mysqld
sudo systemctl status mysqld
```

### 11.2 Important Paths

| Description    | Path                |
| -------------- | ------------------- |
| Config file    | /etc/my.cnf         |
| Data directory | /var/lib/mysql      |
| Log file       | /var/log/mysqld.log |

### 11.3 References

- MySQL: https://www.mysql.com
- Docs: https://dev.mysql.com/doc/
- Rocky Linux: https://rockylinux.org

---

## 12. Document History

| Version | Date       | Changes                   | Author   |
| ------- | ---------- | ------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for MySQL | xianzhan |

---
