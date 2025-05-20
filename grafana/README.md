# Grafana Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Grafana is an open-source analytics and visualization platform used to monitor metrics from various sources such as Prometheus, InfluxDB, MySQL, and more. It provides beautiful, customizable dashboards for time-series data. This guide outlines the steps to install and configure Grafana on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                                    | Recommended |
| -------- | ---------------------------------------------------------- | ----------- |
| CPU      | 1 core                                                     | 2 cores     |
| RAM      | 1 GB                                                       | 2 GB+       |
| Disk     | 5 GB                                                       | 10 GB+      |
| Network  | Required for dashboard access and data source connectivity |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Dependencies: wget, curl, yum-utils, firewalld, systemd
- Web browser: For accessing the Grafana UI

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Add Grafana Repository

```bash
sudo tee /etc/yum.repos.d/grafana.repo <<EOF
[grafana]
name=Grafana OSS
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
EOF
```

### 3.3 Install Grafana

```bash
sudo dnf install -y grafana
```

## 4. Configure Firewall

Open default Grafana port (3000):
```bash
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Enable and Start Grafana

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

### 5.2 Access Grafana Web Interface

Open your browser and go to:
```bash
http://<your-server-ip>:3000
```
Default credentials:
- Username: admin
- Password: admin
You will be prompted to change the password on first login.

---

## 6. Post-Installation Configuration

### 6.1 Add Data Source

Navigate to:
```bash
Configuration → Data Sources → Add data source
```
Choose your data source (e.g., Prometheus), and provide the connection details.

### 6.2 Create Dashboard

Go to:
```bash
Create → Dashboard → Add new panel
```
Choose metrics, visualization type, and save as a dashboard.

### 6.3 User and Organization Management (Optional)

Navigate to:
```bash
Server Admin → Users / Orgs
```
Add users, assign roles, or manage organizations.

---

## 7. Backup & Restore

### 7.1 Backup

Stop Grafana:
```bash
sudo systemctl stop grafana-server
```
Backup config and data:
```bash
sudo tar -czvf grafana-backup.tar.gz /var/lib/grafana /etc/grafana
```
Start Grafana:
```bash
sudo systemctl start grafana-server
```

### 7.2 Restore

Stop Grafana:
```bash
sudo systemctl stop grafana-server
```
Restore backup:
```bash
sudo tar -xzvf grafana-backup.tar.gz -C /
```
Ensure correct permissions:
```bash
sudo chown -R grafana:grafana /var/lib/grafana /etc/grafana
```
Start Grafana:
```bash
sudo systemctl start grafana-server
```

---

## 8. Logging & Monitoring

- Check logs:
```bash
journalctl -u grafana-server
```
- Log file:
```bash
/var/log/grafana/grafana.log
```
- Check status:
```bash
sudo systemctl status grafana-server
```

---

## 9. Troubleshooting

| Issue                    | Resolution                                              |
| ------------------------ | ------------------------------------------------------- |
| Cannot access UI         | Ensure Grafana is running and firewall allows port 3000 |
| Login fails              | Verify username/password, reset if necessary            |
| Data source errors       | Check connectivity and authentication details           |
| No graphs/metrics        | Verify data source is working (e.g., Prometheus)        |
| Crashes or memory issues | Check logs, increase system resources                   |

---

## 10. Uninstallation (if needed)

```bash
sudo systemctl stop grafana-server
sudo dnf remove grafana
sudo rm -rf /etc/grafana /var/lib/grafana /var/log/grafana
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
sudo systemctl start grafana-server
sudo systemctl stop grafana-server
sudo systemctl restart grafana-server
sudo systemctl status grafana-server
```

### 11.2 Important Paths

| Description    | Path                     |
| -------------- | ------------------------ |
| Config file    | /etc/grafana/grafana.ini |
| Data directory | /var/lib/grafana         |
| Log file       | /var/log/grafana         |

### 11.3 References

- Grafana: https://grafana.com
- Documentation: https://grafana.com/docs/

---

## 12. Document History

| Version | Date       | Changes                     | Author   |
| ------- | ---------- | --------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for Grafana | xianzhan |

---
