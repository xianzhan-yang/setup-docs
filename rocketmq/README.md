# RocketMQ Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Apache RocketMQ is a distributed messaging and streaming platform with low latency, high performance, and reliability. This guide provides step-by-step instructions to install and configure RocketMQ on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                                            | Recommended |
| -------- | ------------------------------------------------------------------ | ----------- |
| CPU      | 2 cores                                                            | 4 cores     |
| RAM      | 2 GB                                                               | 4 GB+       |
| Disk     | 5 GB                                                               | 10 GB+      |
| Network  | Required for communication between NameServer, Broker, and clients |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Java: OpenJDK 8 or 11
- SELinux: Permissive or configured for RocketMQ
- Dependencies: wget, unzip, firewalld, systemd, gcc, git, maven

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

### 3.3 Create RocketMQ User and Directories 

```bash
sudo useradd -M -s /sbin/nologin rocketmq
sudo mkdir -p /opt/rocketmq
sudo chown -R rocketmq:rocketmq /opt/rocketmq
```

### 3.4 Download and Extract RocketMQ

```bash
cd /opt
sudo wget https://downloads.apache.org/rocketmq/5.1.4/rocketmq-all-5.1.4-bin-release.zip
sudo unzip rocketmq-all-5.1.4-bin-release.zip -d rocketmq
sudo chown -R rocketmq:rocketmq /opt/rocketmq
```

---

## 4. Configure Firewall

RocketMQ uses the following default ports:
```bash
sudo firewall-cmd --permanent --add-port=9876/tcp  # NameServer
sudo firewall-cmd --permanent --add-port=10911/tcp # Broker
sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Start NameServer

```bash
sudo su - rocketmq
cd /opt/rocketmq/bin
nohup sh mqnamesrv &
```
Check NameServer logs:
```bash
tail -f /opt/rocketmq/logs/rocketmqlogs/namesrv.log
```

### 5.2 Start Broker

```bash
nohup sh mqbroker -n localhost:9876 &
```
Check Broker logs:
```bash
tail -f /opt/rocketmq/logs/rocketmqlogs/broker.log
```

---

## 6. Post-Installation Configuration

### 6.1 Optional: Set Environment Variables

Add the following to /etc/profile.d/rocketmq.sh:
```bash
export ROCKETMQ_HOME=/opt/rocketmq
export PATH=$PATH:$ROCKETMQ_HOME/bin
```
```bash
source /etc/profile.d/rocketmq.sh
```

### 6.2 Optional: Create a Systemd Service

Create a systemd unit for NameServer:
```bash
sudo vi /etc/systemd/system/rocketmq-nameserver.service
```
Paste:
```bash
[Unit]
Description=RocketMQ NameServer
After=network.target

[Service]
User=rocketmq
ExecStart=/opt/rocketmq/bin/mqnamesrv
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Then enable and start:
```bash
sudo systemctl daemon-reexec
sudo systemctl enable rocketmq-nameserver
sudo systemctl start rocketmq-nameserver
```
Repeat for mqbroker as needed.

---

## 7. Backup & Restore

### 7.1 Backup

Stop services:
```bash
sudo systemctl stop rocketmq-nameserver
sudo systemctl stop rocketmq-broker
```
Backup directory:
```bash
sudo tar -czvf rocketmq-backup.tar.gz /opt/rocketmq
```
Start services:
```bash
sudo systemctl start rocketmq-nameserver
sudo systemctl start rocketmq-broker
```

### 7.2 Restore

Stop services:
```bash
sudo systemctl stop rocketmq-nameserver
sudo systemctl stop rocketmq-broker
```
Restore:
```bash
sudo tar -xzvf rocketmq-backup.tar.gz -C /
```
Ensure permissions:
```bash
sudo chown -R rocketmq:rocketmq /opt/rocketmq
```
Start services:
```bash
sudo systemctl start rocketmq-nameserver
sudo systemctl start rocketmq-broker
```

---

## 8. Logging & Monitoring

- Log files: /opt/rocketmq/logs/rocketmqlogs/
- Monitor NameServer and Broker logs:
```bash
tail -f /opt/rocketmq/logs/rocketmqlogs/namesrv.log
tail -f /opt/rocketmq/logs/rocketmqlogs/broker.log
```
- Optional: Use RocketMQ Dashboard or Prometheus exporters

---

## 9. Troubleshooting

| Issue                    | Resolution                                         |
| ------------------------ | -------------------------------------------------- |
| NameServer not starting  | Check logs in namesrv.log                          |
| Broker fails to register | Ensure NameServer is running and reachable         |
| Port not accessible      | Check firewalld rules, confirm services are active |
| Message loss or delay    | Review disk I/O, broker memory settings            |
| Dashboard not available  | Confirm port 8080 is open and app is running       |

---

## 10. Uninstallation (if needed)

```bash
sudo systemctl stop rocketmq-nameserver
sudo systemctl stop rocketmq-broker
sudo systemctl disable rocketmq-nameserver
sudo systemctl disable rocketmq-broker
sudo rm -rf /opt/rocketmq /etc/systemd/system/rocketmq-*.service
sudo userdel rocketmq
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
sh mqnamesrv
sh mqbroker -n localhost:9876
sh mqadmin topicList -n localhost:9876
sh mqadmin updateTopic -n localhost:9876 -t TestTopic -c DefaultCluster
```

### 11.2 Important Paths

| Description   | Path                             |
| ------------- | -------------------------------- |
| RocketMQ home | /opt/rocketmq                    |
| Config files  | /opt/rocketmq/conf               |
| Log files     | /opt/rocketmq/logs/rocketmqlogs/ |

### 11.3 References

- RocketMQ: https://rocketmq.apache.org
- Docs: https://rocketmq.apache.org/docs/
- GitHub: https://github.com/apache/rocketmq
- Dashboard: https://github.com/apache/rocketmq-dashboard

---

## 12. Document History

| Version | Date       | Changes                      | Author   |
| ------- | ---------- | ---------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for RocketMQ | xianzhan |

---
