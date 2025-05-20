# LVS Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Linux Virtual Server (LVS) is a high-performance and highly available load balancing solution built into the Linux kernel. It is commonly used to distribute network traffic across multiple servers using IPVS (IP Virtual Server). This guide provides step-by-step instructions to install and configure LVS on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                     | Recommended |
| -------- | --------------------------- | ----------- |
| CPU      | 2 cores                     | 4 cores     |
| RAM      | 1 GB                        | 2 GB+       |
| Disk     | 5 GB                        | 10 GB+      |
| Network  | Required for load balancing |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Kernel: With IPVS support (enabled by default)
- Tools: ipvsadm, iproute, net-tools
- SELinux: Permissive or configured appropriately

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Install IPVS Tools

```bash
sudo dnf install -y ipvsadm
```
Verify installation:
```bash
ipvsadm --version
```

---

## 4. Configure LVS (NAT Mode Example)

### 4.1 Enable IP Forwarding

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```
Make it permanent:
```bash
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 4.2 Configure Virtual Service (VIP)

Example: Load balancing TCP traffic on port 80 to two real servers.
```bash
sudo ipvsadm -A -t 192.168.1.100:80 -s rr
sudo ipvsadm -a -t 192.168.1.100:80 -r 192.168.1.101:80 -m
sudo ipvsadm -a -t 192.168.1.100:80 -r 192.168.1.102:80 -m
```
Options:
- -s rr: Round-robin scheduling
- -m: NAT forwarding method

### 4.3 View Configuration

```bash
sudo ipvsadm -Ln
```

---

## 5. Persist Configuration

### 5.1 Install ipvsadm Service

Create a systemd service or use a script to restore rules at boot.
Example save:
```bash
sudo ipvsadm-save > /etc/sysconfig/ipvsadm
```
Enable restore at boot:
```bash
sudo systemctl enable ipvsadm
```

---

## 6. Firewall Configuration

Allow traffic to VIP port (e.g., port 80):
```bash
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload
```

---

## 7. Initial Setup

### 7.1 Test Load Balancing

From a client:
```bash
curl http://192.168.1.100
```
Repeat and verify response alternates between real servers.

### 7.2 Monitor LVS Table

```bash
watch -n 1 ipvsadm -Ln
```

---

## 8. Post-Installation Configuration

### 8.1 Scheduler Algorithms

You may choose other schedulers:
- rr: Round-robin
- wrr: Weighted round-robin
- lc: Least connection
Example:
```bash
ipvsadm -A -t 192.168.1.100:80 -s lc
```

### 8.2 Health Checking (Optional)

LVS does not provide native health checking. Consider pairing with Keepalived.

---

## 9. Backup & Restore

### 9.1 Backup

```bash
sudo ipvsadm-save > ipvs-backup.conf
```

### 9.2 Restore

```bash
sudo ipvsadm-restore < ipvs-backup.conf
```

---

## 10. Troubleshooting

| Issue                   | Resolution                                    |
| ----------------------- | --------------------------------------------- |
| VIP not accessible      | Check IP forwarding, firewall, network routes |
| Traffic not balanced    | Check real server status and configuration    |
| Rules lost after reboot | Use ipvsadm-save and enable boot restore      |
| SELinux issues          | Set SELinux to permissive for testing         |

---

## 11. Uninstallation (if needed)

```bash
sudo dnf remove ipvsadm
sudo systemctl disable ipvsadm
sudo rm -f /etc/sysconfig/ipvsadm
```

---

## 12. Appendix

### 12.1 Useful Commands

```bash
ipvsadm -Ln
ipvsadm -C         # Clear all rules
ipvsadm-save       # Save current rules
ipvsadm-restore    # Restore rules
```

### 12.2 Important Paths

| Description     | Path                   |
| --------------- | ---------------------- |
| Config backup   | /etc/sysconfig/ipvsadm |
| IPVS admin tool | /usr/sbin/ipvsadm      |

### 12.3 References

- LVS: http://www.linuxvirtualserver.org/
- ipvsadm man page: man ipvsadm

---

## 13. Document History

| Version | Date       | Changes                 | Author   |
| ------- | ---------- | ----------------------- | -------- |
| 1.0     | 2025-05-20 | Initial version for LVS | xianzhan |

---
