# Terraform Deployment Guide (Rocky Linux 9)

---

## 1. Overview

Terraform is an open-source infrastructure as code (IaC) tool created by HashiCorp. It enables users to define and provision data center infrastructure using a high-level configuration language. This guide provides step-by-step instructions to install and configure Terraform on Rocky Linux 9.

---

## 2. System Requirements

### 2.1 Hardware Requirements

| Resource | Minimum                                             | Recommended |
| -------- | --------------------------------------------------- | ----------- |
| CPU      | 1 core                                              | 2 cores     |
| RAM      | 512 MB                                              | 1 GB+       |
| Disk     | 100 MB                                              | 500 MB+     |
| Network  | Required for provider downloads and remote backends |             |

### 2.2 Software Requirements

- OS: Rocky Linux 9 (x86_64)
- Dependencies: wget, unzip, curl, sudo

---

## 3. Installation Steps 

### 3.1 Update the System

```bash
sudo dnf update -y
```

### 3.2 Add HashiCorp Repository

```bash
sudo dnf install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
```

### 3.3 Install Terraform

```bash
sudo dnf install -y terraform
```

### 3.4 Verify Installation

```bash
terraform version
```

---

## 4. Configure Firewall

Terraform itself does not require any specific ports. However, ensure outbound internet access is available for provider plugin downloads and remote backends.

---

## 5. Initial Setup

### 5.1 Create Terraform Project Directory

```bash
mkdir -p ~/terraform-demo
cd ~/terraform-demo
```

### 5.2 Sample Terraform File

```bash
# main.tf
provider "local" {}

resource "local_file" "example" {
  content  = "Hello, Terraform!"
  filename = "${path.module}/hello.txt"
}
```

### 5.3 Initialize Project

```bash
terraform init
```

### 5.4 Apply Configuration

```bash
terraform apply
```
Approve when prompted.

---

## 6. Post-Installation Configuration

### 6.1 Set Environment Variables (Optional)

You may want to configure cloud provider credentials or Terraform backend settings using environment variables.

Example (for AWS):
```bash
export AWS_ACCESS_KEY_ID=your_key
export AWS_SECRET_ACCESS_KEY=your_secret
```

### 6.2 Install Additional Plugins (Providers)

Terraform automatically installs required providers during terraform init.

---

## 7. Backup & Restore

### 7.1 Backup

Backup Terraform project directory:
```bash
tar -czvf terraform-backup.tar.gz ~/terraform-demo
```

### 7.2 Restore

```bash
tar -xzvf terraform-backup.tar.gz -C ~/
```
Ensure credentials or provider configurations are updated if necessary.

---

## 8. Logging & Monitoring

- Terraform does not produce service logs by default.
- Use TF_LOG for debugging:
```bash
export TF_LOG=DEBUG
terraform apply
```
- Logs will print to stdout or can be redirected to a file with TF_LOG_PATH.

---

## 9. Troubleshooting

| Issue                    | Resolution                                                  |
| ------------------------ | ----------------------------------------------------------- |
| Provider install failure | Check internet connection or proxy settings                 |
| Invalid credentials      | Ensure environment variables or provider config are correct |
| Command not found        | Check if Terraform is in `$PATH`                            |
| Errors in `.tf` file     | Run `terraform validate` to check syntax                    |

---

## 10. Uninstallation (if needed)

```bash
sudo dnf remove terraform
sudo rm -rf ~/.terraform.d ~/terraform-demo
```

---

## 11. Appendix

### 11.1 Useful Commands

```bash
terraform init
terraform plan
terraform apply
terraform destroy
terraform fmt
terraform validate
```

### 11.2 Important Paths

| Description     | Path                         |
| --------------- | ---------------------------- |
| User config dir | \~/.terraform.d              |
| Plugin download | .terraform/                  |
| Executable path | /usr/bin/terraform (default) |

### 11.3 References

- Terraform: https://www.terraform.io
- Docs: https://developer.hashicorp.com/terraform/docs

---

## 12. Document History

| Version | Date       | Changes                       | Author   |
| ------- | ---------- | ----------------------------- | -------- |
| 1.0     | 2025-05-18 | Initial version for Terraform | xianzhan |

---
