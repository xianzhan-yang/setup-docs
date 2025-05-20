# Kubernetes Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Kubernetes is a powerful open-source system for automating deployment, scaling, and management of containerized applications. This guide provides step-by-step instructions to install and configure a single-node Kubernetes cluster using kubeadm on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                               | Recommended |
| -------- | ----------------------------------------------------- | ----------- |
| CPU      | 2 cores                                               | 4+ cores    |
| RAM      | 4 GB                                                  | 8 GB+       |
| Disk     | 20 GB                                                 | 40 GB+ SSD  |
| Network  | Required for pod communication and cluster operations |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Container Runtime: containerd (recommended)
- SELinux: Permissive or configured
- Required Tools: kubeadm, kubelet, kubectl, iptables, curl, firewalld, systemd

---

## 3. Installation Steps

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Disable Swap

```bash
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
```

### 3.3 Configure Kernel Modules and Sysctl

```bash
sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

### 3.4 Install containerd

```bash
sudo dnf install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl enable --now containerd
```

### 3.5 Add Kubernetes Repository

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el9-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg 
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

### 3.6 Install Kubernetes Components

```bash
sudo dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```

---

## 4. Initialize Cluster

### 4.1 Pull Images

```bash
sudo kubeadm config images pull
```

### 4.2 Initialize Master Node

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

### 4.3 Configure kubectl for Current User

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 5. Install Pod Network

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

## 6. Configure Firewall

Open necessary ports:
```bash
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=2379-2380/tcp
sudo firewall-cmd --permanent --add-port=10250-10252/tcp
sudo firewall-cmd --permanent --add-port=30000-32767/tcp
sudo firewall-cmd --reload
```

---

## 7. Node Configuration (Optional for Worker Nodes)

Use the kubeadm join command from master output to join additional nodes:
```bash
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```

---

## 8. Post-Installation Configuration

### 8.1 Enable Bash Completion

```bash
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

### 8.2 Install Metrics Server (Optional)

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

## 9. Backup & Restore

### 9.1 Backup (Master Node)

```bash
sudo ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key
```

### 9.2 Restore (Disaster Recovery)

```bash
sudo systemctl stop kubelet
sudo ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
--data-dir=/var/lib/etcd-from-backup
# Update etcd manifest to point to new data dir
sudo systemctl start kubelet
```

---

## 10. Logging & Monitoring

- System logs: journalctl -u kubelet
- Cluster logs: Use kubectl logs
- Monitoring: Install Prometheus, Grafana (see separate guides)

---

## 11. Troubleshooting

| Issue                 | Resolution                                              |
| --------------------- | ------------------------------------------------------- |
| kubeadm init fails    | Check logs, ensure swap is off, verify network settings |
| kubelet not running   | Check `journalctl -u kubelet`, verify containerd        |
| Nodes not Ready       | Check network plugin and firewall settings              |
| Pods stuck in Pending | Check `kubectl describe pod <pod>` for events           |

---

## 12. Uninstallation (if needed)

```bash
sudo kubeadm reset
sudo dnf remove -y kubeadm kubectl kubelet containerd
sudo rm -rf ~/.kube /etc/kubernetes /var/lib/etcd /var/lib/kubelet
```

---

## 13. Appendix

### 13.1 Useful Commands

```bash
kubectl get nodes
kubectl get pods -A
kubectl describe pod <pod>
kubectl apply -f <file>.yaml
```

### 13.2 Important Paths

| Description  | Path                  |
| ------------ | --------------------- |
| Kube config  | \~/.kube/config       |
| Kubelet logs | journalctl -u kubelet |
| etcd data    | /var/lib/etcd         |

### 13.3 References

- Kubernetes: https://kubernetes.io
- kubeadm Docs: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/

---

## 14. Document History

| Version | Date       | Changes                        | Author   |
| ------- | ---------- | ------------------------------ | -------- |
| 1.0     | 2025-05-18 | Initial version for Kubernetes | xianzhan |

---
