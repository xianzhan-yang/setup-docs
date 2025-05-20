# Ansible Deployment Guide (Rocky Linux 9)

---

## 1. Overview 

Ansible is an open-source automation tool used for configuration management, application deployment, and orchestration. It uses simple YAML-based playbooks and requires no agent on managed nodes. This guide provides step-by-step instructions to install and configure Ansible on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                         | Recommended |
| -------- | ----------------------------------------------- | ----------- |
| CPU      | 1 core                                          | 2 cores     |
| RAM      | 1 GB                                            | 2 GB+       |
| Disk     | 2 GB                                            | 5 GB+       |
| Network  | Required for SSH-based control of managed nodes |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Python: Preinstalled (required by Ansible)
- SSH: Required for remote node management
- EPEL repository: Required for installing Ansible

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Enable EPEL Repository

```bash
sudo dnf install -y epel-release
```

### 3.3 Install Ansible

```bash
sudo dnf install -y ansible
```
Verify installation:
```bash
ansible --version
```

---

## 4. Configure SSH Access to Managed Nodes

Ensure SSH access is set up between the control node (where Ansible is installed) and the managed nodes.

### 4.1 Generate SSH Key (if not existing)

```bash
ssh-keygen -t rsa -b 4096
```

### 4.2 Copy Key to Managed Node(s)

```bash
ssh-copy-id user@managed-node-ip
```
Test connection:
```bash
ssh user@managed-node-ip
```

---

## 5. Initial Setup

### 5.1 Inventory File

Create a simple inventory file /etc/ansible/hosts:
```bash
[webservers]
192.168.1.100
192.168.1.101

[dbservers]
192.168.1.102
```
Test connectivity:
```bash
ansible all -m ping
```

---

## 6. Post-Installation Configuration

### 6.1 Ansible Configuration File

Default config file: /etc/ansible/ansible.cfg

You can customize:
- Inventory location
- Default module path
- Logging
- Privilege escalation (become)
- SSH arguments
Example:
```bash
[defaults]
inventory = /etc/ansible/hosts
remote_user = your_user
host_key_checking = False
```

---

## 7. Backup & Restore

### 7.1 Backup

Back up important configuration and playbooks:
```bash
tar -czvf ansible-backup.tar.gz /etc/ansible /home/your_user/playbooks
```

### 7.2 Restore

Unpack backup:
```bash
tar -xzvf ansible-backup.tar.gz -C /
```
Ensure correct permissions:
```bash
chown -R your_user:your_user /home/your_user/playbooks
```

---

## 8. Logging & Monitoring

- Ansible does not log by default. To enable logging:
Edit /etc/ansible/ansible.cfg:
```bash
[defaults]
log_path = /var/log/ansible.log
```
- To view logs:
```bash
cat /var/log/ansible.log
```

---

## 9. Troubleshooting

| Issue              | Resolution                                           |
| ------------------ | ---------------------------------------------------- |
| SSH errors         | Ensure SSH key is copied and permissions are correct |
| Module not found   | Check Ansible version and module support             |
| Host unreachable   | Validate IP, firewall, and SSH                       |
| YAML errors        | Check indentation and syntax                         |
| Permissions denied | Use `--become` or configure `become` in playbooks    |

---

## 10. Uninstallation (if needed)

```bash
sudo dnf remove -y ansible
sudo rm -rf /etc/ansible /var/log/ansible.log
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
ansible all -m ping
ansible all -a "uptime"
ansible-playbook site.yml
```

### 11.2 Important Paths

| Description    | Path                     |
| -------------- | ------------------------ |
| Config file    | /etc/ansible/ansible.cfg |
| Inventory file | /etc/ansible/hosts       |
| Log file       | /var/log/ansible.log     |

### 11.3 References

- Ansible: https://www.ansible.com
- Docs: https://docs.ansible.com

---

## 12. Document History

| Version | Date       | Changes                     | Author   |
| ------- | ---------- | --------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for Ansible | xianzhan |

---
