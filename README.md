# DevOps Deployments

This repository provides a collection of deployment guides for common DevOps and infrastructure tools. Each directory contains step-by-step instructions, configuration tips, and references for setting up the respective software in production-like environments.

Whether you're building a self-hosted DevOps stack, testing tools in a lab environment, or documenting deployment procedures for your team, this repository aims to be a practical and modular resource.

---

## 📦 Supported Tools

| Tool          | Description                                        |
|---------------|----------------------------------------------------|
| [Ansible](ansible/README.md)       | IT automation and configuration management tool |
| [Docker](docker/README.md)         | Platform for containerizing applications         |
| [Elasticsearch](elasticsearch/README.md) | Search and analytics engine                |
| [Git](git/README.md)               | Distributed version control system               |
| [GitLab](gitlab/README.md)         | Git-based DevOps platform with CI/CD             |
| [Grafana](grafana/README.md)       | Metrics visualization and dashboarding tool      |
| [Jenkins](jenkins/README.md)       | CI/CD automation server                          |
| [Kibana](kibana/README.md)         | UI for querying and visualizing Elasticsearch data |
| [Kubernetes](kubernetes/README.md) | Container orchestration system                   |
| [Linux](linux/README.md)           | Core system setup and essential commands         |
| [Logstash](logstash/README.md)     | Logs and event data processing pipeline          |
| [MySQL](mysql/README.md)           | Relational database system                       |
| [Nacos](nacos/README.md)           | Service registry and configuration platform      |
| [Nginx](nginx/README.md)           | Web server and reverse proxy                     |
| [Prometheus](prometheus/README.md) | Metrics collection and alerting system           |
| [Redis](redis/README.md)           | In-memory data structure store                   |
| [RocketMQ](rocketmq/README.md)     | Distributed messaging and streaming platform     |
| [Terraform](terraform/README.md)   | Infrastructure as Code (IaC) provisioning tool   |
| [Zabbix](zabbix/README.md)         | Enterprise-grade monitoring and alerting         |

## 📁 Directory Structure

```bash
devops-deployments/
├── ansible/
├── docker/
├── elasticsearch/
├── git/
├── gitlab/
├── grafana/
├── jenkins/
├── kibana/
├── kubernetes/
├── linux/
├── logstash/
├── mysql/
├── nacos/
├── nginx/
├── prometheus/
├── redis/
├── rocketmq/
├── terraform/
├── zabbix/
└── README.md
```

## 🛠️ Usage

- Browse to the folder of the tool you want to deploy.
- Follow the instructions in its `README.md`.
- Most guides are tailored for Linux environments and focus on manual or semi-automated deployment using shell, systemd, or containerized runtimes.

## 🎯 Audience

This repository is for:
- DevOps engineers deploying and maintaining infrastructure
- Developers building local or test environments
- Students and professionals learning DevOps tooling
- Teams documenting internal deployment procedures

## 📜 License

This project is licensed under the terms of the [LICENSE](LICENSE) file.

---

> ✅ Contributions and suggestions are welcome! Feel free to open issues or submit pull requests to improve or expand the guides.
