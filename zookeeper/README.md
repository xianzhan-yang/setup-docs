# Zookeeper Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Apache ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. This guide provides step-by-step instructions to install and configure Zookeeper on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                               | Recommended |
| -------- | ------------------------------------- | ----------- |
| CPU      | 2 cores                               | 4 cores     |
| RAM      | 2 GB                                  | 4 GB+       |
| Disk     | 10 GB                                 | 20 GB+ SSD  |
| Network  | Required for distributed coordination |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Java: OpenJDK 21
- Dependencies: wget, tar, systemd

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Install Java

```bash
sudo dnf install -y java-21-openjdk
```
Verify installation:
```bash
java -version
```

### 3.3 Create Zookeeper User

```bash
sudo useradd -m -s /bin/bash zookeeper
```

### 3.4 Download and Extract Zookeeper

```bash
cd /opt
sudo wget https://downloads.apache.org/zookeeper/zookeeper-3.9.2/apache-zookeeper-3.9.2-bin.tar.gz
sudo tar -xvzf apache-zookeeper-3.9.2-bin.tar.gz
sudo mv apache-zookeeper-3.9.2-bin zookeeper
sudo chown -R zookeeper:zookeeper /opt/zookeeper
```

---

## 4. Configure Zookeeper

### 4.1 Create Configuration File

```bash
sudo mkdir -p /opt/zookeeper/data
sudo tee /opt/zookeeper/conf/zoo.cfg <<EOF
tickTime=2000
dataDir=/opt/zookeeper/data
clientPort=2181
maxClientCnxns=60
EOF
sudo chown -R zookeeper:zookeeper /opt/zookeeper
```

---

## 5. Enable and Start Zookeeper

### 5.1 Create systemd Service

```bash
sudo tee /etc/systemd/system/zookeeper.service <<EOF
[Unit]
Description=Apache Zookeeper
After=network.target

[Service]
Type=simple
User=zookeeper
ExecStart=/opt/zookeeper/bin/zkServer.sh start-foreground
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

### 5.2 Start and Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable zookeeper
sudo systemctl start zookeeper
```
Check status:
```bash
sudo systemctl status zookeeper
```

---

## 6. Configure Firewall

ZooKeeper uses port 2181 by default.
```bash
sudo firewall-cmd --permanent --add-port=2181/tcp
sudo firewall-cmd --reload
```

---

## 7. Initial Setup

### 7.1 Verify Zookeeper Is Running

```bash
echo ruok | nc localhost 2181
```
Expected response:
```bash
imok
```

---

## 8. Post-Installation Configuration

### 8.1 Configure for Cluster (Optional)

To run Zookeeper in cluster mode (ensemble), update zoo.cfg:
```bash
server.1=zk1:2888:3888
server.2=zk2:2888:3888
server.3=zk3:2888:3888
```
Also create a myid file in each node’s dataDir:
```bash
echo 1 | sudo tee /opt/zookeeper/data/myid
```

---

## 9. Backup & Restore

## 9.1 Backup

Stop Zookeeper:
```bash
sudo systemctl stop zookeeper
```
Backup data:
```bash
sudo tar -czvf zookeeper-backup.tar.gz /opt/zookeeper/data
```
Start Zookeeper:
```bash
sudo systemctl start zookeeper
```

### 9.2 Restore

Stop Zookeeper:
```bash
sudo systemctl stop zookeeper
```
Restore data:
```bash
sudo tar -xzvf zookeeper-backup.tar.gz -C /
```
Ensure permissions:
```bash
sudo chown -R zookeeper:zookeeper /opt/zookeeper/data
```
Start Zookeeper:
```bash
sudo systemctl start zookeeper
```

---

## 10. Logging & Monitoring

- Log file: /opt/zookeeper/zookeeper.out
- Monitor service:
```bash
sudo systemctl status zookeeper
```
- Optional: Use zkCli.sh to interact with Zookeeper

---

## 11. Troubleshooting

| Issue                    | Resolution                                            |
| ------------------------ | ----------------------------------------------------- |
| Zookeeper not starting   | Check logs at `/opt/zookeeper/zookeeper.out`          |
| Port 2181 not accessible | Check firewall, systemctl status                      |
| Connection refused       | Ensure ZooKeeper is running and clientPort is correct |
| Slow response            | Review tickTime and monitor server resources          |

---

## 12. Uninstallation (if needed)

```bash
sudo systemctl stop zookeeper
sudo systemctl disable zookeeper
sudo rm -rf /opt/zookeeper /etc/systemd/system/zookeeper.service
sudo userdel -r zookeeper
```

---

## 13. Appendix

### 13.1 Useful Commands

```bash
zkCli.sh
echo ruok | nc localhost 2181
```

### 13.2 Important Paths

| Description    | Path                         |
| -------------- | ---------------------------- |
| Zookeeper home | /opt/zookeeper               |
| Config file    | /opt/zookeeper/conf/zoo.cfg  |
| Data dir       | /opt/zookeeper/data          |
| Log file       | /opt/zookeeper/zookeeper.out |

### 13.3 References

- Zookeeper: https://zookeeper.apache.org
- Rocky Linux: https://rockylinux.org

---

## 14. Document History

| Version | Date       | Changes                       | Author   |
| ------- | ---------- | ----------------------------- | -------- |
| 1.0     | 2025-05-19 | Initial version for Zookeeper | xianzhan |

---
