# Tomcat Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Apache Tomcat is an open-source implementation of the Java Servlet, JavaServer Pages, Java Expression Language, and Java WebSocket technologies. This guide provides step-by-step instructions to install and configure Tomcat on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                        | Recommended |
| -------- | ---------------------------------------------- | ----------- |
| CPU      | 1 core                                         | 2 cores     |
| RAM      | 1 GB                                           | 2 GB+       |
| Disk     | 2 GB                                           | 5 GB+       |
| Network  | Required for HTTP access and remote management |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Java: OpenJDK 21
- Dependencies: wget, tar, firewalld, systemd

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

### 3.3 Create Tomcat User

```bash
sudo useradd -m -U -d /opt/tomcat -s /bin/false tomcat
```

### 3.4 Download and Extract Tomcat

```bash
cd /tmp
wget https://downloads.apache.org/tomcat/tomcat-10/v10.1.24/bin/apache-tomcat-10.1.24.tar.gz
sudo mkdir -p /opt/tomcat
sudo tar -xzf apache-tomcat-10.1.24.tar.gz -C /opt/tomcat --strip-components=1
```

### 3.5 Set Permissions

```bash
sudo chown -R tomcat: /opt/tomcat
sudo sh -c 'chmod +x /opt/tomcat/bin/*.sh'
```

---

## 4. Configure Systemd Service

Create systemd service unit:
```bash
sudo tee /etc/systemd/system/tomcat.service <<EOF
[Unit]
Description=Apache Tomcat
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```
Enable and start Tomcat:
```bash
sudo systemctl daemon-reload
sudo systemctl enable tomcat
sudo systemctl start tomcat
```
Check status:
```bash
sudo systemctl status tomcat
```

---

## 5. Configure Firewall

Tomcat listens on port 8080 by default:
```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

---

## 6. Initial Setup

### 6.1 Access Tomcat

Open your browser and navigate to:
```bash
http://<your-server-ip>:8080
```
You should see the Tomcat welcome page.

### 6.2 Configure Users (Manager Access)

Edit tomcat-users.xml:
```bash
sudo nano /opt/tomcat/conf/tomcat-users.xml
```
Add:
```bash
<role rolename="manager-gui"/>
<role rolename="admin-gui"/>
<user username="admin" password="yourpassword" roles="manager-gui,admin-gui"/>
```
Restart Tomcat:
```bash
sudo systemctl restart tomcat
```

---

## 7. Post-Installation Configuration

### 7.1 Secure Manager Apps

Restrict access to Manager and Host Manager:
```bash
sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml
```
Comment or modify access restrictions:
```bash
<!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve"
       allow="127\.\d+\.\d+\.\d+|::1" /> -->
```
Repeat for:
```bash
/opt/tomcat/webapps/host-manager/META-INF/context.xml
```
Restart Tomcat after changes.

---

## 8. Backup & Restore

### 8.1 Backup

Stop Tomcat:
```bash
sudo systemctl stop tomcat
```
Backup directory:
```bash
sudo tar -czvf tomcat-backup.tar.gz /opt/tomcat
```
Start Tomcat:
```bash
sudo systemctl start tomcat
```

### 8.2 Restore

Stop Tomcat:
```bash
sudo systemctl stop tomcat
```
Restore:
```bash
sudo tar -xzvf tomcat-backup.tar.gz -C /
```
Ensure permissions:
```bash
sudo chown -R tomcat:tomcat /opt/tomcat
```
Start Tomcat:
```bash
sudo systemctl start tomcat
```

---

## 9. Logging & Monitoring

- Log files: /opt/tomcat/logs/
- Monitor service:
```bash
sudo systemctl status tomcat
```
- Access logs:
```bash 
tail -f /opt/tomcat/logs/catalina.out
```

---

## 10. Troubleshooting

| Issue                       | Resolution                                  |
| --------------------------- | ------------------------------------------- |
| Tomcat not starting         | Check logs in `/opt/tomcat/logs/`           |
| Port 8080 not accessible    | Check firewall settings                     |
| 403 Forbidden (manager app) | Update `tomcat-users.xml` and `context.xml` |
| Deployment fails            | Check war file structure and logs           |
| High memory usage           | Tune JVM options in `setenv.sh`             |

---

## 11. Uninstallation (if needed)

```bash
sudo systemctl stop tomcat
sudo systemctl disable tomcat
sudo rm -rf /opt/tomcat /etc/systemd/system/tomcat.service
sudo userdel -r tomcat
```

---

## 12. Appendix

### 12.1 Useful Commands

```bash
sudo systemctl start tomcat
sudo systemctl stop tomcat
sudo systemctl restart tomcat
sudo systemctl status tomcat
```

### 12.2 Important Paths

| Description | Path             |
| ----------- | ---------------- |
| Tomcat home | /opt/tomcat      |
| Config file | /opt/tomcat/conf |
| Logs        | /opt/tomcat/logs |

### 12.3 References

- Tomcat: https://tomcat.apache.org

---

## 13. Document History

| Version | Date       | Changes                    | Author   |
| ------- | ---------- | -------------------------- | -------- |
| 1.0     | 2025-05-19 | Initial version for Tomcat | xianzhan |

---
