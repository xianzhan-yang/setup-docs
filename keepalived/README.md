# Keepalived Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Keepalived is a routing software that provides simple and robust facilities for load balancing and high-availability. It is commonly used to achieve virtual IP failover via VRRP. This guide provides step-by-step instructions to install and configure Keepalived on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                          | Recommended |
| -------- | -------------------------------- | ----------- |
| CPU      | 1 core                           | 2 cores     |
| RAM      | 512 MB                           | 1 GB+       |
| Disk     | 100 MB                           | 500 MB+     |
| Network  | Required for VRRP and IP binding |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Dependencies: iproute, net-tools, firewalld, systemd
- SELinux: Permissive or properly configured

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Install Keepalived

```bash
sudo dnf install -y keepalived
```
Verify installation:
```bash
keepalived -v
```

---

## 4. Configure Keepalived

### 4.1 Backup Default Configuration

```bash
sudo cp /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.bak
```

### 4.2 Edit Configuration File

Edit /etc/keepalived/keepalived.conf:
```bash
sudo nano /etc/keepalived/keepalived.conf
```
Example (MASTER node):
```bash
vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass secretpass
    }
    virtual_ipaddress {
        192.168.1.100
    }
}
```
Replace:
- interface: your active network interface
- 192.168.1.100: your desired virtual IP

---

## 5. Enable and Start Keepalived

```bash
sudo systemctl enable keepalived
sudo systemctl start keepalived
```
Check status:
```bash
sudo systemctl status keepalived
```

## 6. Configure Firewall

Allow VRRP protocol (IP protocol number 112):
```bash
sudo firewall-cmd --permanent --add-rich-rule='rule protocol value="112" accept'
sudo firewall-cmd --reload
```

---

## 7. Initial Setup

### 7.1 Check Virtual IP

On MASTER node:
```bash
ip addr show | grep 192.168.1.100
```
You should see the virtual IP assigned.

### 7.2 Test Failover

- Stop Keepalived on MASTER:
```bash
sudo systemctl stop keepalived
```
- Observe the virtual IP move to BACKUP node (priority < 100)

---

## 8. Post-Installation Configuration

### 8.1 Log Configuration

Keepalived logs via syslog. To view:
```bash
sudo journalctl -u keepalived
```
To enable file logging, edit /etc/rsyslog.conf or configure a custom log file via keepalived.conf.

### 8.2 Add Health Checks (Optional)

Add track_script to monitor services (e.g., Nginx):
```bash
vrrp_script chk_nginx {
    script "/usr/bin/pgrep nginx"
    interval 2
    weight -10
}

vrrp_instance VI_1 {
    ...
    track_script {
        chk_nginx
    }
}
```

---

## 9. Backup & Restore

### 9.1 Backup

```bash
sudo cp /etc/keepalived/keepalived.conf keepalived.conf.bak
```

### 9.2 Restore

```bash
sudo cp keepalived.conf.bak /etc/keepalived/keepalived.conf
sudo systemctl restart keepalived
```

---

## 10. Troubleshooting

| Issue                   | Resolution                                                   |
| ----------------------- | ------------------------------------------------------------ |
| Keepalived not starting | Check config file syntax, view logs                          |
| VIP not showing         | Check interface name, VRRP ID conflicts                      |
| No failover             | Ensure both nodes are properly configured and on same subnet |
| Logs missing            | Check systemd journal or configure rsyslog                   |

---

## 11. Uninstallation (if needed)

```bash
sudo systemctl stop keepalived
sudo dnf remove keepalived
sudo rm -rf /etc/keepalived
```

---

## 12. Appendix

### 12.1 Useful Commands

```bash
sudo systemctl start keepalived
sudo systemctl stop keepalived
sudo systemctl restart keepalived
sudo systemctl status keepalived
```

### 12.2 Important Paths

| Description   | Path                            |
| ------------- | ------------------------------- |
| Config file   | /etc/keepalived/keepalived.conf |
| Log (journal) | `journalctl -u keepalived`      |
| Backup config | `/etc/keepalived/*.bak`         |

---

### 12.3 References

- Keepalived: https://keepalived.org

---

## 13. Document History

| Version | Date       | Changes                        | Author   |
| ------- | ---------- | ------------------------------ | -------- |
| 1.0     | 2025-05-20 | Initial version for Keepalived | xianzhan |

---
