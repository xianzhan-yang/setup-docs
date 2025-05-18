# Prometheus Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Prometheus is an open-source monitoring and alerting toolkit designed for reliability and scalability. It collects metrics from configured targets at given intervals, evaluates rule expressions, and can trigger alerts if needed. This guide provides step-by-step instructions to install and configure Prometheus on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                      | Recommended              |
| -------- | -------------------------------------------- | ------------------------ |
| CPU      | 2 cores                                      | 4 cores                  |
| RAM      | 2 GB                                         | 4 GB+                    |
| Disk     | 10 GB                                        | 20 GB+ (SSD recommended) |
| Network  | Required for data scraping and web interface |                          |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Dependencies: wget, curl, tar, firewalld, systemd

---

## 3. Installation Steps 

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Create Prometheus User and Directories

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus
```

### 3.3 Download and Install Prometheus

```bash
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar -xvf prometheus-2.52.0.linux-amd64.tar.gz
cd prometheus-2.52.0.linux-amd64
```
Copy binaries:
```bash
sudo cp prometheus promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
```
Copy configuration files:
```bash
sudo cp -r consoles/ console_libraries/ /etc/prometheus/
sudo cp prometheus.yml /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

## 4. Configure Firewall

Prometheus uses port 9090:
```bash
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Create systemd Service File

Create file: /etc/systemd/system/prometheus.service
```bash
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
Reload systemd and start Prometheus:
```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```
Check status:
```bash
sudo systemctl status prometheus
```

---

## 6. Post-Installation Configuration

### 6.1 Verify Web Access

Open in browser:
```bash
http://<your-server-ip>:9090
```

### 6.2 Edit Targets in prometheus.yml

Example:
```bash
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```
Restart Prometheus to apply changes:
```bash
sudo systemctl restart prometheus
```

---

## 7. Backup & Restore

### 7.1 Backup

Stop Prometheus:
```bash
sudo systemctl stop prometheus
```
Backup data:
```bash
sudo tar -czvf prometheus-backup.tar.gz /var/lib/prometheus
```
Start Prometheus:
```bash
sudo systemctl start prometheus
```

### 7.2 Restore

Stop Prometheus:
```bash
sudo systemctl stop prometheus
```
Restore:
```bash
sudo tar -xzvf prometheus-backup.tar.gz -C /
```
Ensure permissions:
```bash
sudo chown -R prometheus:prometheus /var/lib/prometheus
```
Start Prometheus:
```bash
sudo systemctl start prometheus
```

---

## 8. Logging & Monitoring

- Prometheus logs:
```bash
journalctl -u prometheus
```
- Web UI:
```bash
http://<your-server-ip>:9090
```
- Target Status:
Navigate to Status → Targets in UI.

---

## 9. Troubleshooting

| Issue                    | Resolution                                                                        |
| ------------------------ | --------------------------------------------------------------------------------- |
| Prometheus not starting  | Check logs via `journalctl -xe` or `systemctl status prometheus`                  |
| Port 9090 not accessible | Check firewalld, confirm Prometheus is running                                    |
| No targets appearing     | Ensure correct `targets` in `prometheus.yml`                                      |
| “permission denied”      | Check ownership of data and config directories                                    |
| High disk usage          | Adjust retention settings in service file with `--storage.tsdb.retention.time=7d` |

---

## 10. Uninstallation (if needed)

```bash
sudo systemctl stop prometheus
sudo systemctl disable prometheus
sudo rm -rf /usr/local/bin/prometheus /usr/local/bin/promtool
sudo rm -rf /etc/prometheus /var/lib/prometheus
sudo rm /etc/systemd/system/prometheus.service
sudo userdel prometheus
sudo systemctl daemon-reload
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
sudo systemctl start prometheus
sudo systemctl stop prometheus
sudo systemctl restart prometheus
sudo systemctl status prometheus
```

### 11.2 Important Paths

| Description     | Path                           |
| --------------- | ------------------------------ |
| Binary location | /usr/local/bin/prometheus      |
| Config file     | /etc/prometheus/prometheus.yml |
| Data directory  | /var/lib/prometheus            |
| Log access      | journalctl -u prometheus       |

### 11.3 References

- Prometheus: https://prometheus.io
- Prometheus Docs: https://prometheus.io/docs/
- Rocky Linux: https://rockylinux.org

---

## 12. Document History

| Version | Date       | Changes                        | Author   |
| ------- | ---------- | ------------------------------ | -------- |
| 1.0     | 2025-05-17 | Initial version for Prometheus | xianzhan |

---
