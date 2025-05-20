# Kafka Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Apache Kafka is a distributed event streaming platform capable of handling trillions of events per day. This guide provides step-by-step instructions to install and configure Kafka on Rocky Linux 9 using the official binaries.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                               | Recommended |
| -------- | ------------------------------------- | ----------- |
| CPU      | 2 cores                               | 4+ cores    |
| RAM      | 4 GB                                  | 8 GB+       |
| Disk     | 20 GB                                 | 100 GB+ SSD |
| Network  | Required for inter-node communication |             |


### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Java: OpenJDK 21
- Dependencies: wget, curl, tar, systemd
- Zookeeper (Required, can run bundled or standalone)

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

### 3.3 Create Kafka User

```bash
sudo useradd -m -s /bin/bash kafka
```

### 3.4 Download and Extract Kafka

```bash
cd /opt
sudo wget https://downloads.apache.org/kafka/3.7.0/kafka_2.13-3.7.0.tgz
sudo tar -xvzf kafka_2.13-3.7.0.tgz
sudo mv kafka_2.13-3.7.0 kafka
sudo chown -R kafka:kafka /opt/kafka
```

---

## 4. Configure Systemd Services

### 4.1 Zookeeper Service

Create a systemd unit:
```bash
sudo tee /etc/systemd/system/zookeeper.service <<EOF
[Unit]
Description=Apache Zookeeper
After=network.target

[Service]
Type=simple
User=kafka
ExecStart=/opt/kafka/bin/zookeeper-server-start.sh /opt/kafka/config/zookeeper.properties
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
EOF
```

### 4.2 Kafka Service

Create a systemd unit:
```bash
sudo tee /etc/systemd/system/kafka.service <<EOF
[Unit]
Description=Apache Kafka
After=zookeeper.service

[Service]
Type=simple
User=kafka
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
EOF
```

---

## 5. Enable and Start Services

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable zookeeper
sudo systemctl enable kafka
sudo systemctl start zookeeper
sudo systemctl start kafka
```
Check status:
```bash
sudo systemctl status kafka
```

---

## 6. Configure Firewall

Kafka default port is 9092. Zookeeper uses 2181.
```bash
sudo firewall-cmd --permanent --add-port=9092/tcp
sudo firewall-cmd --permanent --add-port=2181/tcp
sudo firewall-cmd --reload
```

---

## 7. Initial Setup

### 7.1 Create a Topic

```bash
/opt/kafka/bin/kafka-topics.sh --create --topic test --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

### 7.2 Send a Message

```bash
/opt/kafka/bin/kafka-console-producer.sh --topic test --bootstrap-server localhost:9092
```

### 7.3 Consume a Message

```bash
/opt/kafka/bin/kafka-console-consumer.sh --topic test --from-beginning --bootstrap-server localhost:9092
```

---

## 8. Post-Installation Configuration

### 8.1 Configure Broker ID and Advertised Listeners

Edit /opt/kafka/config/server.properties:
```bash
broker.id=0
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://<your-server-ip>:9092
log.dirs=/tmp/kafka-logs
```

---

## 9. Backup & Restore

### 9.1 Backup

Stop Kafka:
```bash
sudo systemctl stop kafka
```
Backup logs:
```bash
sudo tar -czvf kafka-backup.tar.gz /tmp/kafka-logs
```
Start Kafka:
```bash
sudo systemctl start kafka
```

### 9.2 Restore

Stop Kafka:
```bash
sudo systemctl stop kafka
```
Restore logs:
```bash
sudo tar -xzvf kafka-backup.tar.gz -C /
```
Start Kafka:
```bash
sudo systemctl start kafka
```

---

## 10. Logging & Monitoring

- Kafka log: /opt/kafka/logs/server.log
- Zookeeper log: /opt/kafka/logs/zookeeper.log
- Monitor with external tools: Prometheus, Grafana, Kafka Manager, etc.

---

## 11. Troubleshooting

| Issue                    | Resolution                                         |
| ------------------------ | -------------------------------------------------- |
| Kafka not starting       | Check logs in `/opt/kafka/logs/server.log`         |
| Port 9092 not accessible | Check firewall, confirm `advertised.listeners`     |
| Topic not found          | Recreate topic, check ZooKeeper connection         |
| Slow performance         | Tune JVM, adjust partitions and retention policies |

---

## 12. Uninstallation (if needed)

```bash
sudo systemctl stop kafka zookeeper
sudo systemctl disable kafka zookeeper
sudo rm -rf /opt/kafka /etc/systemd/system/kafka.service /etc/systemd/system/zookeeper.service
sudo userdel -r kafka
```

---

## 13. Appendix

### 13.1 Useful Commands

```bash
systemctl start kafka
systemctl stop kafka
kafka-topics.sh --list --bootstrap-server localhost:9092
```

### 13.2 Important Paths

| Description   | Path                          |
| ------------- | ----------------------------- |
| Kafka home    | /opt/kafka                    |
| Server log    | /opt/kafka/logs/server.log    |
| Zookeeper log | /opt/kafka/logs/zookeeper.log |

### 13.3 References

- Kafka: https://kafka.apache.org
- Kafka Docs: https://kafka.apache.org/documentation/

---

## 14. Document History

| Version | Date       | Changes                   | Author   |
| ------- | ---------- | ------------------------- | -------- |
| 1.0     | 2025-05-19 | Initial version for Kafka | xianzhan |

---
