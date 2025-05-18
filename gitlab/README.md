# GitLab Deployment Guide (Rocky Linux 9)

---

## 1. Overview

GitLab is a powerful open-source DevOps platform for source code management, CI/CD pipelines, project planning, and more. This document guides you through installing and configuring GitLab Community Edition on Rocky Linux 9 to ensure stable operation.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum  | Recommended     |
| -------- | -------- | --------------- |
| CPU      | 2 cores  | 4 cores or more |
| Memory   | 4 GB     | 8 GB or more    |
| Disk     | 20 GB    | 40 GB or more   |
| Network  | Required | Required        |

### 2.2 Software Requirements

- Operating System: Rocky Linux 9 (x86_64)
- Packages: curl, policycoreutils, openssh-server, postfix
- SELinux: Set to permissive or properly configured
- Access: Root privileges and internet access

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Install Required Dependencies

```bash
sudo dnf install -y curl policycoreutils-python-utils openssh-server postfix
sudo systemctl enable --now sshd
sudo systemctl enable --now postfix
```

### 3.3 Add GitLab Repository and Install

```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
sudo dnf install -y gitlab-ce
```

### 3.4 Configure GitLab

Edit configuration file:
```bash
sudo vim /etc/gitlab/gitlab.rb
```
Set your external URL:
```bash
external_url 'http://<server-ip>'
```
Apply configuration:
```bash
sudo gitlab-ctl reconfigure
```

---

## 4. Configure Firewall

Allow HTTP, HTTPS, and SSH:
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-port=22/tcp
sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Access GitLab Web UI

Open a browser and navigate to:
```bash
http://<server-ip>
```

### 5.2 Log In

Log in with:
- Username: root
- Password: 
```bash
sudo cat /etc/gitlab/initial_root_password 
```

---

## 6. Post-Installation Configuration

### 6.1 Configure Email Notification

Edit SMTP settings in:
```bash
sudo vim /etc/gitlab/gitlab.rb
```
Example configuration:
```bash
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.example.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "user@example.com"
gitlab_rails['smtp_password'] = "password"
gitlab_rails['smtp_domain'] = "example.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
```
Reconfigure GitLab:
```bash
sudo gitlab-ctl reconfigure
```

### 6.2 Set System Message (Banner)

Use a custom banner message for system status or warnings:
```bash
sudo gitlab-rails console
```
```bash
ApplicationSetting.last.update_attribute(:login_text, "GitLab CE on Rocky Linux 9")
```

### 6.3 Configure CI/CD Runners (Optional)

Register runners via:
```bash
sudo gitlab-runner register
```
Set executor type, tags, and GitLab instance URL during registration.

---

## 7. Backup and Restore

### 7.1 Backup

Run backup:
```bash
sudo gitlab-backup create
```
Backups are saved in:
```bash
/var/opt/gitlab/backups/
```
Optional: Stop GitLab before backup for consistency.
```bash
sudo gitlab-ctl stop
sudo gitlab-backup create
sudo gitlab-ctl start
```

### 7.2 Restore

Restore from a specific backup:
```bash
sudo gitlab-ctl stop puma
sudo gitlab-ctl stop sidekiq
sudo gitlab-backup restore BACKUP=timestamp
sudo gitlab-ctl start
```
Ensure permissions:
```bash
sudo gitlab-ctl reconfigure
```

---

## 8. Maintenance and Monitoring

### 8.1 Log Access

GitLab logs are located at:
```bash
/var/log/gitlab/
```
Tail logs (e.g., Nginx or GitLab):
```bash
sudo tail -f /var/log/gitlab/gitlab-rails/production.log
```

### 8.2 Monitoring

Built-in monitoring with Prometheus and Grafana available via:
```bash
http://<server-ip>/-/metrics
```
You may also install custom monitoring tools or GitLab Prometheus exporter.

---

## 9. Troubleshooting

| Issue                  | Solution                                             | Notes                                |
| ---------------------- | ---------------------------------------------------- | ------------------------------------ |
| Web UI not accessible  | Check services, ports, SELinux, and logs             | `gitlab-ctl status`, firewall-cmd    |
| SMTP email not sending | Review `/etc/gitlab/gitlab.rb` and mail logs         | Restart GitLab after changes         |
| CI/CD runner offline   | Check runner token, registration, and GitLab version | Use `gitlab-runner verify`           |
| Password lost          | Reset via `gitlab-rails console`                     | Use: `User.find_by_username('root')` |
| 500/502 errors         | Review logs, check NGINX, Unicorn, or memory issues  | `sudo gitlab-ctl tail`               |

---

## 10. Uninstallation (if needed)

Stop and remove GitLab:
```bash
sudo gitlab-ctl stop
sudo dnf remove -y gitlab-ce
sudo rm -rf /etc/gitlab /var/opt/gitlab /var/log/gitlab
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
sudo gitlab-ctl start
sudo gitlab-ctl stop
sudo gitlab-ctl restart
sudo gitlab-ctl reconfigure
sudo gitlab-rake gitlab:check
sudo gitlab-ctl tail
```

### 11.2 Important Paths

| Description    | Path                    |
| -------------- | ----------------------- |
| GitLab config  | /etc/gitlab             |
| Data directory | /var/opt/gitlab         |
| Logs           | /var/log/gitlab         |
| Backups        | /var/opt/gitlab/backups |

### 11.3 Useful Links

- GitLab Official: https://about.gitlab.com
- Documentation: https://docs.gitlab.com
- Rocky Linux: https://rockylinux.org

---

## 12. Document History

| Version | Date       | Changes                     | Author   |
| ------- | ---------- | --------------------------- | -------- |
| 1.0     | 2025-05-17 | Initial version for Rocky 9 | xianzhan |

---
