# Docker Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Docker is a platform for developing, shipping, and running applications in containers. This guide provides step-by-step instructions to install and configure Docker on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                        | Recommended |
| -------- | ---------------------------------------------- | ----------- |
| CPU      | 2 cores                                        | 4 cores     |
| RAM      | 2 GB                                           | 4 GB+       |
| Disk     | 10 GB                                          | 20 GB+      |
| Network  | Required for package downloads and image pulls |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- SELinux: Permissive or configured for container use
- Dependencies: dnf, curl, wget, firewalld (optional)

---

## 3. Installation Steps

### 3.1 Update the System 

```bash
sudo dnf update -y
```

### 3.2 Install Required Dependencies

```bash
sudo dnf install -y dnf-plugins-core
```

### 3.3 Add Docker Repository

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### 3.4 Install Docker

```bash
sudo dnf install -y docker-ce docker-ce-cli containerd.io
```

### 3.5 Enable and Start Docker

```bash
sudo systemctl enable --now docker
```
Check status:
```bash
sudo systemctl status docker
```

---

## 4. Configure Firewall (Optional)

If you're running containers that expose ports (e.g., web servers):
```bash
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Verify Docker Installation

```bash
docker --version
```

### 5.2 Run Test Container

```bash
sudo docker run hello-world
```

### 5.3 Manage Docker as Non-root User (Optional)

```bash
sudo usermod -aG docker $USER
```

---

## 6. Post-Installation Configuration

### 6.1 Configure Docker to Start on Boot

```bash
sudo systemctl enable docker
```

### 6.2 Configure SELinux (Optional)

If SELinux is enforcing and causes issues:
```bash
sudo setenforce 0
# or permanently:
sudo sed -i 's/^SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config
```

### 6.3 Configure Docker Daemon (Optional)

Edit /etc/docker/daemon.json to set mirror, log limits, etc.
Example:
```bash
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
```
Restart Docker:
```bash
sudo systemctl restart docker
```

---

## 7. Backup & Restore

### 7.1 Backup

```bash
docker run --rm -v my_volume:/volume -v $(pwd):/backup alpine \
  tar czvf /backup/my_volume_backup.tar.gz -C /volume . 
```

### 7.2 Restore

```bash
docker run --rm -v my_volume:/volume -v $(pwd):/backup alpine \
  tar xzvf /backup/my_volume_backup.tar.gz -C /volume
```

---

## 8. Logging & Monitoring

- Docker daemon logs:
```bash
journalctl -u docker
```
- Container logs:
```bash
docker logs <container_id>
```
- Monitor running containers:
```bash
docker ps
docker stats
```

---

## 9. Troubleshooting

| Issue                               | Resolution                                                |
| ----------------------------------- | --------------------------------------------------------- |
| Docker not starting                 | Check `systemctl status docker`, look in `journalctl -xe` |
| Permission denied                   | Add user to `docker` group or run with `sudo`             |
| Images not pulling                  | Check internet connection, DNS settings                   |
| SELinux blocking containers         | Set SELinux to permissive or add appropriate policies     |
| Containers can’t access the network | Check firewalld and DNS configuration                     |

---

## 10. Uninstallation (if needed)

```bash
sudo systemctl stop docker
sudo dnf remove docker-ce docker-ce-cli containerd.io
sudo rm -rf /var/lib/docker /etc/docker
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
docker ps -a
docker images
docker exec -it <container_id> /bin/bash
docker stop <container_id>
docker rm <container_id>
```

### 11.2 Important Paths

| Description     | Path                        |
| --------------- | --------------------------- |
| Docker root dir | /var/lib/docker             |
| Config file     | /etc/docker/daemon.json     |
| Systemd service | systemctl start/stop docker |

### 11.3 References

- Docker: https://www.docker.com
- Docker Docs: https://docs.docker.com

---

## 12. Document History

| Version | Date       | Changes                    | Author   |
| ------- | ---------- | -------------------------- | -------- |
| 1.0     | 2025-05-17 | Initial version for Docker | xianzhan |

---
