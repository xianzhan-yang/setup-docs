# Linux Initialization Guide (Rocky Linux 9)

---

## 1. Overview

This guide provides step-by-step instructions to initialize and configure a fresh Rocky Linux 9 system. It includes tasks such as system updates, user creation, SSH hardening, firewall configuration, time synchronization, and basic performance tuning.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                             | Recommended |
| -------- | ----------------------------------- | ----------- |
| CPU      | 1 core                              | 2 cores     |
| RAM      | 1 GB                                | 2 GB+       |
| Disk     | 10 GB                               | 20 GB+      |
| Network  | Required for updates and SSH access |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Access: Root or sudo privileges
- Tools: wget, curl, vim, firewalld, chrony, sudo

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Install Essential Tools

```bash
sudo dnf install -y vim wget curl git net-tools unzip zip lsof telnet tree
```

---

## 4. Configure Firewall

### 4.1 Enable and Configure firewalld

```bash
sudo systemctl enable firewalld
sudo systemctl start firewalld
```

### 4.2 Allow SSH, HTTP, and HTTPS Ports

```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Create New User with Sudo

```bash
sudo useradd -m devops
sudo passwd devops
sudo usermod -aG wheel devops
```

### 5.2 Configure SSH

Edit SSH configuration to disable root login and allow password-based authentication:
```bash
sudo vim /etc/ssh/sshd_config
```
Add or modify the following lines:
```bash
PermitRootLogin no
PasswordAuthentication yes
Port 22
```
Restart the SSH service:
```bash
sudo systemctl restart sshd
```

---

## 6. Post-Installation Configuration

### 6.1 Set Hostname

```bash
sudo hostnamectl set-hostname myserver.example.com
```

### 6.2 Set Timezone and Enable NTP

```bash
sudo timedatectl set-timezone Asia/Shanghai
sudo dnf install -y chrony
sudo systemctl enable --now chronyd
```

### 6.3 Set SELinux to Permissive (Optional)

```bash
sudo setenforce 0
sudo sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config
```

---

## 7. Backup & Restore

### 7.1 Backup

```bash
sudo tar -czvf etc-backup.tar.gz /etc
```

### 7.2 Restore

```bash
sudo tar -xzvf etc-backup.tar.gz -C /
```

---

## 8. Logging & Monitoring

### 8.1 System Logs

- Access logs: /var/log/messages
- Authentication logs: /var/log/secure

### 8.2 Monitor System Status

To monitor system status:
```bash
journalctl -xe
tail -f /var/log/messages
```

### 8.3 Optional Monitoring Tools

You may install monitoring tools like htop, nmon, or glances for better system health tracking.

---

## 9. Troubleshooting

| Issue                    | Resolution                                                       |
| ------------------------ | ---------------------------------------------------------------- |
| SSH connection refused   | Check if the SSH service is running and firewall is allowing SSH |
| Hostname not updated     | Reboot the system or use `hostnamectl`                           |
| Time not syncing         | Ensure `chronyd` is running and time settings are correct        |
| Permission denied        | Check user permissions and groups                                |
| SELinux blocking service | Set SELinux to permissive or configure appropriate policies      |

---

## 10. Uninstallation (if needed)

Reverting the system to pre-init state is typically not done. However, you can remove specific configurations or packages:

```bash
sudo userdel -r devops
sudo dnf remove <package-name>
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
sudo reboot
sudo systemctl status <service-name>
sudo dnf install <package-name>
sudo firewall-cmd --list-all
sudo visudo
```

### 11.2 Important Paths

| Description           | Path                  |
| --------------------- | --------------------- |
| SSH configuration     | /etc/ssh/sshd\_config |
| Hostname              | /etc/hostname         |
| SELinux configuration | /etc/selinux/config   |
| System logs           | /var/log/messages     |

### 11.3 References

- Rocky Linux: https://rockylinux.org
- SELinux Documentation: https://wiki.centos.org/HowTos/SELinux
- Chrony: https://chrony.tuxfamily.org

---

## 12. Document History

| Version | Date       | Changes                     | Author   |
| ------- | ---------- | --------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for Rocky 9 | xianzhan |

---
