# Logstash Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Logstash is a powerful, flexible data processing pipeline that ingests data from multiple sources, transforms it, and sends it to a specified output like Elasticsearch. This guide provides step-by-step instructions to install and configure Logstash on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                | Recommended |
| -------- | -------------------------------------- | ----------- |
| CPU      | 2 cores                                | 4+ cores    |
| RAM      | 4 GB                                   | 8 GB+       |
| Disk     | 20 GB                                  | 40 GB+ SSD  |
| Network  | Required for data ingestion and output |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Java: Bundled with Logstash
- SELinux: Permissive or configured
- Dependencies: wget, curl, systemd, firewalld

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Import Elasticsearch GPG Key

```bash
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

### 3.3 Add Logstash Repository

```bash
sudo tee /etc/yum.repos.d/logstash.repo <<EOF
[logstash]
name=Elastic repository for Logstash
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```

### 3.4 Install Logstash

```bash
sudo dnf install -y logstash
```

---

## 4. Configuration

### 4.1 Create Configuration File

Example input/output pipeline:
```bash
sudo vim /etc/logstash/conf.d/simple.conf
```
```bash
input {
  stdin { }
}
output {
  stdout { codec => rubydebug }
}
```

---

## 5. Enable and Start Logstash

```bash
sudo systemctl enable logstash
sudo systemctl start logstash
```
Check status:
```bash
sudo systemctl status logstash
```

---

## 6. Configure Firewall

Logstash uses port 5044 for Beats input (optional):
```bash
sudo firewall-cmd --permanent --add-port=5044/tcp
sudo firewall-cmd --reload
```

---

## 7. Initial Setup

### 7.1 Test Pipeline

Run test pipeline:
```bash
sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/simple.conf
```
Type text into the terminal to see JSON output.

---

## 8. Post-Installation Configuration

### 8.1 Directory Structure

- Configuration files: /etc/logstash/
- Pipelines: /etc/logstash/conf.d/
- Logs: /var/log/logstash/

### 8.2 Use with Elasticsearch

Example Elasticsearch output config:
```bash
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logstash-%{+YYYY.MM.dd}"
  }
}
```

---

## 9. Backup & Restore

### 9.1 Backup

```bash
sudo tar -czvf logstash-config-backup.tar.gz /etc/logstash
```

### 9.2 Restore

```bash
sudo tar -xzvf logstash-config-backup.tar.gz -C /
```
Restart service:
```bash
sudo systemctl restart logstash
```

---

## 10. Logging & Monitoring

- Log file: /var/log/logstash/logstash-plain.log
- Monitor service:
```bash
sudo systemctl status logstash
```
- Optional: Integrate with Metricbeat or Prometheus Exporter

---

## 11. Troubleshooting

| Issue                  | Resolution                                        |
| ---------------------- | ------------------------------------------------- |
| Logstash won't start   | Check config syntax, run `--config.test_and_exit` |
| Output not received    | Check network, pipeline output settings           |
| Port 5044 inaccessible | Check firewall rules                              |
| High CPU usage         | Reduce filters, increase system resources         |
| Pipeline config errors | Examine `/var/log/logstash/` logs                 |

---

## 12. Uninstallation (if needed)

```bash
sudo systemctl stop logstash
sudo dnf remove logstash
sudo rm -rf /etc/logstash /var/log/logstash
```

---

## 13. Appendix

### 13.1 Useful Commands

```bash
sudo systemctl start logstash
sudo systemctl stop logstash
sudo systemctl restart logstash
sudo systemctl status logstash
```

### 13.2 Important Paths

| Description      | Path                                 |
| ---------------- | ------------------------------------ |
| Config directory | /etc/logstash                        |
| Pipeline configs | /etc/logstash/conf.d/                |
| Log file         | /var/log/logstash/logstash-plain.log |

### 13.3 References

- Logstash: https://www.elastic.co/logstash
- Docs: https://www.elastic.co/guide/en/logstash/current/index.html

---

## 14. Document History

| Version | Date       | Changes                      | Author   |
| ------- | ---------- | ---------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for Logstash | xianzhan |

---
