# Jenkins Deployment Guide (Rocky Linux 9)

## 1. Overview

Jenkins is a widely used open-source automation server designed for continuous integration and continuous delivery (CI/CD). This guide provides step-by-step instructions to install and configure Jenkins on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| CPU      | 2 cores | 4 cores     |
| RAM      | 4 GB    | 8 GB+       |
| Disk     | 20 GB   | 40 GB+      |
| Network  | Required for installation and plugin downloads |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Java: OpenJDK 21
- SELinux: Permissive or configured for Jenkins
- Dependencies: `wget`, `curl`, `firewalld`, `systemd`

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

### 3.3 Add Jenkins Repository

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

### 3.4 Install Jenkins

```bash
sudo dnf install -y jenkins
```

### 3.5 Enable and Start Jenkins

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```
Check status:
```bash
sudo systemctl status jenkins
```

---

## 4. Configure Firewall

Jenkins listens on port 8080 by default.
```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Access Jenkins

Open a browser and go to:
```bash
http://<your-server-ip>:8080
```

### 5.2 Retrieve Initial Admin Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Paste the password into the web interface.

### 5.3 Install Plugins

Choose:
- Install suggested plugins (recommended for most users)
- Or manually select plugins
Wait for installation to complete.

### 5.4 Create Admin User

Set up your first Jenkins admin user.

---

## 6. Post-Installation Configuration

### 6.1 Set Jenkins URL

Go to:
```bash
Manage Jenkins → Configure System → Jenkins URL
```
Set it to:
```bash
http://<your-server-ip>:8080/
```

### 6.2 Configure Email Notifications (Optional)

Add SMTP settings in:
```bash
Manage Jenkins → Configure System → Email Notification
```

### 6.3 Configure Build Tools (e.g. Git, Maven) 

Add tools via:
```bash
Manage Jenkins → Global Tool Configuration
```
Install or configure:
- Git
- Maven
- Gradle
- JDK (optional custom versions)

---

## 7. Backup & Restore

### 7.1 Backup

Stop Jenkins:
```bash
sudo systemctl stop jenkins
```
Backup data directory:
```bash
sudo tar -czvf jenkins-backup.tar.gz /var/lib/jenkins
```
Start Jenkins:
```bash
sudo systemctl start jenkins
```

### 7.2 Restore

Stop Jenkins:
```bash
sudo systemctl stop jenkins
```
Restore:
```bash
sudo tar -xzvf jenkins-backup.tar.gz -C /
```
Ensure permissions:
```bash
sudo chown -R jenkins:jenkins /var/lib/jenkins
```
Start Jenkins:
```bash
sudo systemctl start jenkins
```

---

## 8. Logging & Monitoring

- Log file: /var/log/jenkins/jenkins.log
- Monitor service:
```bash
sudo systemctl status jenkins
```
- Optional: Install monitoring plugins (e.g., Monitoring, Prometheus metrics plugin)

---

## 9. Troubleshooting

| Issue | Resolution |
| --- | --- |
| Jenkins not starting | Check logs in /var/log/jenkins/jenkins.log |
| Port 8080 not accessible | Check firewalld, verify systemctl status jenkins |
| Plugin install failures | Check internet connection, proxy settings, try again |
| Cannot access web UI | Ensure Jenkins is running, correct IP/port used |
| Slow performance | Increase system RAM or CPU, optimize build jobs |

---

## 10. Uninstallation (if needed)

```bash
sudo systemctl stop jenkins
sudo dnf remove jenkins
sudo rm -rf /var/lib/jenkins /etc/sysconfig/jenkins /var/log/jenkins
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
sudo systemctl start jenkins
sudo systemctl stop jenkins
sudo systemctl restart jenkins
sudo systemctl status jenkins
```

### 11.2 Important Paths

| Description  | Path                         |
| ------------ | ---------------------------- |
| Jenkins home | /var/lib/jenkins             |
| Config file  | /etc/sysconfig/jenkins       |
| Service log  | /var/log/jenkins/jenkins.log |

### 11.3 References

- Jenkins: https://www.jenkins.io
- Jenkins Docs: https://www.jenkins.io/doc/

---

## 12. Document History

| Version | Date       | Changes                     | Author   |
| ------- | ---------- | --------------------------- | -------- |
| 1.0     | 2025-05-17 | Initial version for Rocky 9 | xianzhan |

---
