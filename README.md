# Automated High-Availability HAProxy Cluster with Ansible

![Ansible](https://img.shields.io/badge/Ansible-E00A00?style=for-the-badge&logo=ansible&logoColor=white)
![HAProxy](https://img.shields.io/badge/HAProxy-00A651?style=for-the-badge&logo=HAProxy&logoColor=white)
![Keepalived](https://img.shields.io/badge/Keepalived-00599C?style=for-the-badge&logo=linux&logoColor=white)

An automated Ansible playbook designed to provision and configure a High-Availability (HA) load balancing cluster using **HAProxy** and **Keepalived**. This setup eliminates Single Points of Failure (SPOF) by utilizing Virtual IP (VIP) failover logic and supports zero-downtime maintenance via connection draining techniques.

---

## Architecture Overview

The infrastructure consists of two load balancer nodes operating in an Active/Passive setup. Keepalived manages a shared Virtual IP (VIP) and automatically routes traffic to the backup node if the primary master fails.

---

## Key Features

- **Automated Deployment:** Fully automated cluster setup using Ansible roles and playbooks.
- **High Availability & Failover:** Virtual IP management via Keepalived using VRRP protocol.
- **Zero-Downtime Maintenance:** Pre-configured socket inspection allowing smooth **Connection Draining** without interrupting active user sessions.
- **Health Checking:** Automated backend server status monitoring.
- **Statistics Dashboard:** Built-in HAProxy Stats page for real-time traffic monitoring.

---

## Tech Stack & Prerequisites

- **Configuration Management:** Ansible 2.21+
- **Load Balancer:** HAProxy
- **HA Provider:** Keepalived (VRRP)
- **Target OS:** Ubuntu 26.04 LTS / Debian 12 (or RHEL-based distributions)

---

## Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/Nonzxense/ansible-lab.git
cd ansible-lab
```
### 2. Configure Inventory & Variables
Edit the `hosts.yml` file to include your node IP addresses:
```yml
all:
  children:
    ha_cluster:
      vars:
        ansible_user: user
        ansible_python_interpreter: /usr/bin/python3
        ansible_become: true
        ansible_become_method: sudo
      hosts:
        lb1:
          ansible_host: 192.168.1.32
          keepalived_role: MASTER
          keepalived_priority: 101
        lb2:
          ansible_host: 192.168.1.32
          keepalived_role: BACKUP
          keepalived_priority: 100
```
Define the Keepalived password in vault.yml:
```yml
vault_keepalived_pass: "yourpassword"
```
Encrypt your vault:
```bash
ansible-vault encrypt vars/vault.yml
```
### 3. Run the Ansible Playbook

Execute the playbook to install and configure both Keepalived and HAProxy across all nodes:
```bash
ansible-playbook -i hosts.yml playbook.yml --ask-vault-password
```
### Testing & Verification
#### 1. Verify VIP & HAProxy Status
Ensure the Virtual IP is currently bound to the Master node:
```bash
docker exec lb1 ip addr show eth0
```
Access the HAProxy Stats Page in your browser:
[http://192.168.1.32:8404/](http://192.168.1.32:8404/)
### 2. Failover Test
Simulate a failure on the Master node by stopping the Keepalived or HAProxy service:
```bash
docker container stop lb1
```
### 3. Zero-Downtime Connection Draining Test
To drain traffic from a specific backend server before maintenance:
```bash
echo "experimental-mode on; set server http_back/web1 state drain" | sudo socat stdio /var/run/haproxy.sock
```
## License
This project is open-source and available under the MIT License.
