# Docker + Ansible Multi-Node Automation Lab

A small DevOps lab environment using Docker containers as virtual nodes and Ansible for infrastructure automation.

This project demonstrates:

- Multi-container infrastructure setup
- SSH-based remote management
- Ansible inventory configuration
- Ad-hoc command execution
- Playbook automation
- Group-based node management
- Automated nginx deployment

---

# Project Structure

```bash
dockervm/
├── group_node
│   ├── inventory.ini
│   └── playbook.yml
├── inventory.ini
└── playbook.yml
```

---

# Technologies Used

- Docker
- Ansible
- Ubuntu/Linux
- SSH
- Nginx

---

# Infrastructure Overview

| Node | Role |
|------|------|
| node1 | Controller / Managed Node |
| node2 | Managed Node |
| node3 | Managed Node |

All containers communicate through a custom Docker bridge network:

```bash
ansible-net
```

---

# Docker Network Setup

Create the network:

```bash
docker network create ansible-net
```

Create containers:

```bash
docker run -dit --name node1 --network ansible-net python:3.11-slim sleep infinity

docker run -dit --name node2 --network ansible-net python:3.11-slim sleep infinity

docker run -dit --name node3 --network ansible-net python:3.11-slim sleep infinity
```

---

# Container Configuration

Inside each container:

```bash
apt update
apt install -y openssh-server python3 sudo iputils-ping nginx
```

Configure SSH:

```bash
mkdir /var/run/sshd

echo 'root:root123' | chpasswd

sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config

/usr/sbin/sshd
```

---

# Ansible Inventory Example

```ini
[servers]
node1 ansible_host=x.x.x.x ansible_user=root ansible_password=root123
node2 ansible_host=x.x.x.x ansible_user=root ansible_password=root123
node3 ansible_host=x.x.x.x ansible_user=root ansible_password=root123

[servers:vars]
ansible_python_interpreter=/usr/bin/python3
```

---

# Testing Connectivity

Ping all nodes:

```bash
ansible all -i inventory.ini -m ping
```

Example output:

```bash
node1 | SUCCESS => {
    "ping": "pong"
}
```

---

# Ad-Hoc Commands

Create a file on all nodes:

```bash
ansible all -i inventory.ini -m shell -a "touch ansibledockervm"
```

Check nginx:

```bash
ansible servers -i inventory.ini -a "curl localhost"
```

---

# Playbook Automation

Example playbook:

```yaml
---
- name: Install and start nginx
  hosts: servers
  become: true

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Start nginx
      service:
        name: nginx
        state: started
```

Run the playbook:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

---

# Group-Based Automation

The `group_node/` directory demonstrates how to target specific node groups.

Example:

```ini
[master]
node1 ansible_host=172.18.0.2 ansible_user=root ansible_password=root123

[Slave]
node2 ansible_host=172.18.0.3 ansible_user=root ansible_password=root123
node3 ansible_host=172.18.0.4 ansible_user=root ansible_password=root123

[Slave:vars]
ansible_python_interpreter=/usr/bin/python3
```

Playbooks can then target only the `Slave` group.

---

# Learning Outcomes

This project helped demonstrate:

- Infrastructure as Code (IaC)
- Configuration Management
- Container Networking
- SSH Automation
- Service Deployment Automation
- Multi-node orchestration
- Ansible inventory grouping

---

# Future Improvements

- Add Docker Compose
- Use SSH key authentication
- Add Ansible roles
- Add monitoring stack
- Convert containers into full VM simulations
- CI/CD integration

---
# Next Version
- Ready to Run Script
 # Author

Built as a DevOps and Infrastructure Automation practice project.
