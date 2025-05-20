# Kibana Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Kibana is an open-source data visualization and exploration tool used for log and time-series analytics, application monitoring, and operational intelligence use cases. It works closely with Elasticsearch to provide visual insights. This guide provides step-by-step instructions to install and configure Kibana on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                                    | Recommended |
| -------- | ---------------------------------------------------------- | ----------- |
| CPU      | 2 cores                                                    | 4+ cores    |
| RAM      | 4 GB                                                       | 8 GB+       |
| Disk     | 20 GB                                                      | 40 GB+ SSD  |
| Network  | Required for browser access and Elasticsearch connectivity |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Java: Not required (bundled)
- SELinux: Permissive or configured
- Dependencies: wget, curl, systemd, firewalld
- Elasticsearch: Required and must be running

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Add Kibana Repository

```bash
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

sudo tee /etc/yum.repos.d/kibana.repo <<EOF
[kibana]
name=Elastic repository for Kibana
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```

### 3.3 Install Kibana

```bash
sudo dnf install -y kibana
```

---

## 4. Configuration

### 4.1 Edit Configuration File

```bash
sudo vim /etc/kibana/kibana.yml
```
Set the following:
```bash
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://localhost:9200"]
```
Optional: Set logging, authentication, etc.

---

## 5. Enable and Start Kibana

```bash
sudo systemctl enable kibana
sudo systemctl start kibana
```
Check status:
```bash
sudo systemctl status kibana
```

---

## 6. Configure Firewall

Kibana uses port 5601:
```bash
sudo firewall-cmd --permanent --add-port=5601/tcp
sudo firewall-cmd --reload
```

--

## 7. Initial Setup

### 7.1 Access Kibana

Open a browser and go to:
```bash
http://<your-server-ip>:5601
```

### 7.2 Connect to Elasticsearch

Ensure the configured Elasticsearch URL is accessible.

### 7.3 Complete Setup Wizard

- Accept license agreement
- Configure security (optional)
- Set up index patterns and dashboards

---

## 8. Post-Installation Configuration

### 8.1 Enable TLS and Authentication (Optional)

Edit kibana.yml to use HTTPS and configure users via Elasticsearch security features.

### 8.2 Install Plugins (Optional)

```bash
sudo /usr/share/kibana/bin/kibana-plugin list
sudo /usr/share/kibana/bin/kibana-plugin install <plugin-url>
```

---

## 9. Backup & Restore

### 9.1 Backup

```bash
sudo cp -r /etc/kibana kibana-backup
```

### 9.2 Restore

```bash
sudo cp -r kibana-backup/* /etc/kibana/
sudo systemctl restart kibana
```

---

## 10. Logging & Monitoring

- Log file: /var/log/kibana/kibana.log
- Monitor service:
```bash
sudo systemctl status kibana
```
- Optional: Use Metricbeat to monitor Kibana

---

## 11. Troubleshooting

| Issue                           | Resolution                                         |
| ------------------------------- | -------------------------------------------------- |
| Kibana not starting             | Check `/var/log/kibana/kibana.log` for errors      |
| Port 5601 not accessible        | Verify firewall, Kibana config, and systemd status |
| Cannot connect to Elasticsearch | Ensure Elasticsearch is up and accessible          |
| UI loading issues               | Clear browser cache or check Kibana logs           |
| Plugin issues                   | Reinstall plugins or check compatibility           |

---

## 12. Uninstallation (if needed)

```bash
sudo systemctl stop kibana
sudo dnf remove kibana
sudo rm -rf /etc/kibana /var/log/kibana
```

---

## 13. Appendix

### 13.1 Useful Commands

```bash
sudo systemctl start kibana
sudo systemctl stop kibana
sudo systemctl restart kibana
sudo systemctl status kibana
```

### 13.2 Important Paths

| Description      | Path                       |
| ---------------- | -------------------------- |
| Config directory | /etc/kibana                |
| Log file         | /var/log/kibana/kibana.log |
| Binaries         | /usr/share/kibana          |

### 13.3 References

- Kibana: https://www.elastic.co/kibana
- Docs: https://www.elastic.co/guide/en/kibana/current/index.html

---

## 14. Document History

| Version | Date       | Changes                    | Author   |
| ------- | ---------- | -------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for Kibana | xianzhan |

---
