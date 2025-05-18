# Git Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Git is a distributed version control system widely used for tracking changes in source code during software development. This guide provides step-by-step instructions to install and configure Git on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                       | Recommended |
| -------- | --------------------------------------------- | ----------- |
| CPU      | 1 core                                        | 2 cores     |
| RAM      | 512 MB                                        | 1 GB+       |
| Disk     | 500 MB                                        | 1 GB+       |
| Network  | Required for cloning repositories and updates |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Dependencies: curl, wget, openssl, firewalld (optional)

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Install Git (from AppStream)

```bash
sudo dnf install -y git
```
Verify installation:
```bash
git --version
```

---

## 4. Configure Firewall (Optional)

If you're hosting a Git server (e.g., via SSH or HTTP), you may need to open ports:
```bash
# For SSH-based Git access
sudo firewall-cmd --permanent --add-service=ssh

# For HTTP/HTTPS Git access (e.g., GitWeb, Gitea, etc.)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Configure Global Git Settings

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### 5.2 Generate SSH Key (optional, for remote access)

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```
Add your public key (~/.ssh/id_ed25519.pub) to remote Git server (e.g., GitHub, GitLab).

### 5.3 Clone Repository Example

```bash
git clone git@github.com:youruser/your-repo.git
```

---

## 6. Post-Installation Configuration

### 6.1 Enable Useful Git Aliases

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm commit
```

### 6.2 Set Default Editor

```bash
git config --global core.editor "vim"
```

### 6.3 Set Credential Cache (optional)

```bash
git config --global credential.helper cache
```

---

## 7. Backup & Restore

### 7.1 Backup

```bash
tar -czvf my-repos-backup.tar.gz /path/to/git/repositories
```

### 7.2 Restore

```bash
tar -xzvf my-repos-backup.tar.gz -C /desired/restore/location
```

---

## 8. Logging & Monitoring

- View commit history:
```bash
git log
```
- Check Git configuration:
```bash
git config --list
```
- Monitor changes:
```bash
git status
```

---

## 9. Troubleshooting

| Issue                   | Resolution                                           |
| ----------------------- | ---------------------------------------------------- |
| Git not recognized      | Ensure Git is installed and in `$PATH`               |
| Permission denied (SSH) | Check SSH key and agent, verify access rights        |
| Merge conflicts         | Resolve manually, then `git add` and `git commit`    |
| Wrong author info       | Update `user.name` and `user.email` settings         |
| Push denied             | Check remote permissions and branch protection rules |

---

## 10. Uninstallation (if needed)

```bash
sudo dnf remove git
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
git clone <url>
git status
git add .
git commit -m "message"
git push
git pull
```

### 11.2 Important Paths

| Description   | Path                      |
| ------------- | ------------------------- |
| Global config | \~/.gitconfig             |
| Repo config   | /path/to/repo/.git/config |
| SSH key       | \~/.ssh/id\_ed25519.pub   |

### 11.3 References

- Git: https://git-scm.com
- Git Docs: https://git-scm.com/doc
- Rocky Linux: https://rockylinux.org

---

## 12. Document History

| Version | Date       | Changes                 | Author   |
| ------- | ---------- | ----------------------- | -------- |
| 1.0     | 2025-05-17 | Initial version for Git | xianzhan |

---
