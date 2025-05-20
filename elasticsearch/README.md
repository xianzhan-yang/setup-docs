# Elasticsearch Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Elasticsearch is a distributed, RESTful search and analytics engine capable of solving a growing number of use cases. This guide provides step-by-step instructions to install and configure Elasticsearch on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                                | Recommended |
| -------- | ------------------------------------------------------ | ----------- |
| CPU      | 2 cores                                                | 4+ cores    |
| RAM      | 4 GB                                                   | 8 GB+       |
| Disk     | 20 GB                                                  | 50 GB+ SSD  |
| Network  | Required for cluster communication and REST API access |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Java: Bundled with Elasticsearch
- SELinux: Permissive or configured
- Dependencies: wget, curl, firewalld, systemd

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

### 3.3 Add Elasticsearch Repository

```bash
sudo tee /etc/yum.repos.d/elasticsearch.repo <<EOF
[elasticsearch]
name=Elasticsearch repository
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```

### 3.4 Install Elasticsearch

```bash
sudo dnf install -y elasticsearch
```

---

## 4. Configuration

### 4.1 Configure Elasticsearch

Edit the config file:
```bash
sudo vim /etc/elasticsearch/elasticsearch.yml
```
Set at minimum:
```bash
network.host: 0.0.0.0
discovery.type: single-node
```
Optional: Adjust heap size in /etc/elasticsearch/jvm.options

---

## 5. Enable and Start Elasticsearch

```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```
Check status:
```bash
sudo systemctl status elasticsearch
```

## 6. Configure Firewall

Default port: 9200
```bash
sudo firewall-cmd --permanent --add-port=9200/tcp
sudo firewall-cmd --permanent --add-port=9300/tcp
sudo firewall-cmd --reload
```

---

## 7. Initial Setup

### 7.1 Test Elasticsearch

```bash
curl -X GET http://localhost:9200
```

### 7.2 Security Enrollment (Optional for Dev)

For production, configure users and TLS. For quick setup, disable security:
```bash
sudo vim /etc/elasticsearch/elasticsearch.yml
```
Add:
```bash
xpack.security.enabled: false
```
Then restart:
```bash
sudo systemctl restart elasticsearch
```

---

## 8. Post-Installation Configuration

### 8.1 Set Heap Size

Edit:
```bash
sudo vim /etc/elasticsearch/jvm.options
```
Set appropriate heap size (e.g., 2g):
```bash
-Xms2g
-Xmx2g
```

### 8.2 Tune System Settings

Add to /etc/sysctl.conf:
```bash
vm.max_map_count=262144
```
Apply:
```bash
sudo sysctl -w vm.max_map_count=262144
```

---

## 9. Backup & Restore

### 9.1 Backup

Register a repository:
```bash
PUT _snapshot/my_backup
```
Create a snapshot:
```bash
PUT /_snapshot/my_backup/snapshot_1
```

### 9.2 Restore

```bash
POST /_snapshot/my_backup/snapshot_1/_restore
```

---

## 10. Logging & Monitoring

- Log file: /var/log/elasticsearch/elasticsearch.log
- Monitor service:
```bash
sudo systemctl status elasticsearch
```
- Optional: Use Metricbeat or Prometheus Exporter

---

## 11. Troubleshooting

| Issue                     | Resolution                                      |
| ------------------------- | ----------------------------------------------- |
| Service won't start       | Check logs in `/var/log/elasticsearch/`         |
| Port not accessible       | Verify firewall and `network.host` setting      |
| Out of memory             | Adjust heap size or increase system memory      |
| Cluster health red        | Check logs and shard allocation                 |
| Can't access from browser | Ensure port 9200 is open and bound to `0.0.0.0` |

---

## 12. Uninstallation (if needed)

```bash
sudo systemctl stop elasticsearch
sudo dnf remove elasticsearch
sudo rm -rf /etc/elasticsearch /var/lib/elasticsearch /var/log/elasticsearch
```

---

## 13. Appendix

### 13.1 Useful Commands

```bash
sudo systemctl start elasticsearch
sudo systemctl stop elasticsearch
sudo systemctl restart elasticsearch
sudo systemctl status elasticsearch
```

### 13.2 Important Paths

| Description    | Path                                     |
| -------------- | ---------------------------------------- |
| Config file    | /etc/elasticsearch/elasticsearch.yml     |
| Data directory | /var/lib/elasticsearch                   |
| Log file       | /var/log/elasticsearch/elasticsearch.log |

### 13.3 References

- Elasticsearch: https://www.elastic.co/elasticsearch
- Docs: https://www.elastic.co/guide/en/elasticsearch/reference/index.html

---

## 14. Document History

| Version | Date       | Changes                           | Author   |
| ------- | ---------- | --------------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for Elasticsearch | xianzhan |

---
