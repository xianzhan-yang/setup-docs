# Redis Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Redis is a high-performance in-memory key-value store, often used for caching, real-time analytics, message brokering, and session storage. This guide provides step-by-step instructions to install and configure Redis on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                               | Recommended         |
| -------- | ------------------------------------- | ------------------- |
| CPU      | 1 core                                | 2 cores             |
| RAM      | 512 MB                                | 1 GB+               |
| Disk     | 100 MB                                | SSD for performance |
| Network  | Required for remote access (optional) |                     |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- SELinux: Permissive or properly configured
- Packages: curl, firewalld, systemd, gcc (for building from source, optional)

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Install Redis from Default Repository

```bash
sudo dnf install -y redis
```

---

## 4. Configure Firewall (Optional for Remote Access)

```bash
sudo firewall-cmd --permanent --add-port=6379/tcp
sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Enable and Start Redis

```bash
sudo systemctl enable redis
sudo systemctl start redis
```

### 5.2 Check Redis Status

```bash
sudo systemctl status redis
```

---

## 6. Post-Installation Configuration

### 6.1 Test Local Connection

```bash
redis-cli ping
```
Expected output:
```bash
PONG
```

### 6.2 Enable Remote Access (Optional)

Edit the configuration file:
```bash
sudo vi /etc/redis.conf
```
Change:
```bash
bind 127.0.0.1
```
To:
```bash
bind 0.0.0.0
```
And ensure:
```bash
protected-mode no
```
Restart Redis:
```bash
sudo systemctl restart redis
```
Warning: Enabling remote access without authentication can be a security risk. Use with caution in production environments.

---

## 7. Security Hardening (Recommended)

### 7.1 Require Password

Edit /etc/redis.conf and add:
```bash
requirepass StrongRedisPassword123
```
Restart Redis:
```bash
sudo systemctl restart redis
```

### 7.2 Disable Dangerous Commands (Optional)

Comment out or rename commands like FLUSHALL, CONFIG, SHUTDOWN, etc.
Example:
```bash
rename-command FLUSHALL ""
```

---

## 8. Backup & Restore

### 8.1 Backup

Redis stores data in /var/lib/redis (default):
```bash
sudo cp /var/lib/redis/dump.rdb /backup/redis-backup.rdb
```

### 8.2 Restore

Stop Redis:
```bash
sudo systemctl stop redis
```
Restore file:
```bash
sudo cp /backup/redis-backup.rdb /var/lib/redis/dump.rdb
sudo chown redis:redis /var/lib/redis/dump.rdb
```
Start Redis:
```bash
sudo systemctl start redis
```

---

## 9. Logging & Monitoring

- Log file: /var/log/redis/redis.log
- Monitor status:
```bash
sudo systemctl status redis
```
- Optional: Use tools like RedisInsight, Prometheus exporters, or redis-cli --stat

---

## 10. Troubleshooting

| Issue                   | Resolution                               |
| ----------------------- | ---------------------------------------- |
| Redis not starting      | Check logs at `/var/log/redis/redis.log` |
| Cannot connect remotely | Check bind address and firewall          |
| AUTH errors             | Ensure correct password is used          |
| Data not persisting     | Confirm `save` directive in `redis.conf` |
| High memory usage       | Use `maxmemory` and `maxmemory-policy`   |

---

## 11. Uninstallation (if needed)

```bash
sudo systemctl stop redis
sudo dnf remove -y redis
sudo rm -rf /etc/redis /var/lib/redis /var/log/redis
```

---

## 12. Appendix

### 12.1 Useful Commands

```bash
redis-cli
redis-cli -a StrongRedisPassword123
redis-cli monitor
redis-cli info
```

### 12.2 Important Paths

| Description    | Path                     |
| -------------- | ------------------------ |
| Config file    | /etc/redis.conf          |
| Data directory | /var/lib/redis           |
| Log file       | /var/log/redis/redis.log |

### 12.3 References

- Redis: https://redis.io
- Redis Docs: https://redis.io/docs/
- Rocky Linux: https://rockylinux.org

---

## 13. Document History

| Version | Date       | Changes                   | Author   |
| ------- | ---------- | ------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for Redis | xianzhan |

---

