# NGINX Deployment Guide (Rocky Linux 9)

---

## 1. Overview

NGINX is a high-performance web server, reverse proxy, and load balancer. It is widely used for serving static content, proxying requests, and securing applications. This guide provides step-by-step instructions to install and configure NGINX on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                      | Recommended |
| -------- | -------------------------------------------- | ----------- |
| CPU      | 1 core                                       | 2 cores     |
| RAM      | 512 MB                                       | 1 GB+       |
| Disk     | 1 GB                                         | 2 GB+       |
| Network  | Required for web access and reverse proxying |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- SELinux: Permissive or configured
- Packages: curl, firewalld, systemd

---

## 3. Installation Steps 

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Install NGINX

```bash
sudo dnf install -y nginx
```

---

## 4. Configure Firewall

Allow HTTP and optionally HTTPS:
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

---

## 5. Initial Setup

### 5.1 Enable and Start NGINX

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 5.2 Verify Installation

Open your browser and navigate to:
```bash
http://<your-server-ip>
```
You should see the NGINX welcome page.

---

## 6. Post-Installation Configuration

### 6.1 Main Config File

Edit /etc/nginx/nginx.conf to adjust global settings.

### 6.2 Create Server Blocks (Virtual Hosts)

Example: Create a new file /etc/nginx/conf.d/example.com.conf
```bash
server {
    listen 80;
    server_name example.com;

    root /var/www/example.com;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Create the document root:
```bash
sudo mkdir -p /var/www/example.com
echo "<h1>Hello from NGINX</h1>" | sudo tee /var/www/example.com/index.html
sudo chown -R nginx:nginx /var/www/example.com
```
Test configuration:
```bash
sudo nginx -t
```
Reload NGINX:
```bash
sudo systemctl reload nginx
```

---

## 7. SSL Configuration (Optional)

### 7.1 Install Certbot

```bash
sudo dnf install -y epel-release
sudo dnf install -y certbot python3-certbot-nginx
```

### 7.2 Obtain SSL Certificate

```bash
sudo certbot --nginx
```
Follow prompts to obtain and install a free Let's Encrypt certificate.

---

## 8. Logging & Monitoring

- Access log: /var/log/nginx/access.log
- Error log: /var/log/nginx/error.log
- Service status:
```bash
sudo systemctl status nginx
```

---

## 9. Troubleshooting

| Issue                 | Resolution                            |
| --------------------- | ------------------------------------- |
| Cannot access website | Check firewall rules and NGINX status |
| NGINX won’t start     | Run `nginx -t` to check configuration |
| 403 Forbidden error   | Check file permissions in `/var/www`  |
| SSL not working       | Ensure ports 80 and 443 are open      |
| Domain not resolving  | Check DNS records for the domain      |

---

## 10. Uninstallation (if needed)

```bash
sudo systemctl stop nginx
sudo dnf remove -y nginx
sudo rm -rf /etc/nginx /var/log/nginx /var/www/html
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl status nginx
```

### 11.2 Important Paths

| Description      | Path                  |
| ---------------- | --------------------- |
| Config directory | /etc/nginx            |
| Default site     | /usr/share/nginx/html |
| Logs             | /var/log/nginx        |

### 11.3 References

- NGINX: https://nginx.org
- Docs: https://nginx.org/en/docs/
- Rocky Linux: https://rockylinux.org

---

## 12. Document History

| Version | Date       | Changes                   | Author   |
| ------- | ---------- | ------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for NGINX | xianzhan |

---
