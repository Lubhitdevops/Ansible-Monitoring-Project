# Ansible Monitoring Stack Deployment on AWS EC2

## 📋 Project Overview

This project automates the deployment of a comprehensive monitoring stack on AWS EC2 instances using Ansible. The solution installs and configures industry-standard monitoring tools including Prometheus for metrics collection, Grafana for visualization, Alertmanager for alert routing, and Node Exporter for system-level metrics.

## 🏗️ Architecture

### Control Server (EC2 Instance)
- Runs Ansible control node for automation
- Configured with SSH key-based authentication to target nodes
- Hosts playbooks and configuration templates

### Node Server (EC2 Instance)
- Hosts the complete monitoring stack:
  - **Prometheus** - Time-series database and metrics collection (port 9090)
  - **Grafana** - Dashboard and visualization platform (port 3000)
  - **Alertmanager** - Alert routing and management (port 9093)
  - **Node Exporter** - System metrics exporter (port 9100)
- All services configured as systemd services for easy management

## 🚀 Prerequisites

- AWS account with EC2 access
- Two EC2 instances (Amazon Linux 2 recommended)
- Basic understanding of Ansible and SSH key management

## ⚙️ Installation & Configuration

### 1. EC2 Instance Setup
Launch two EC2 instances with the following specifications:
- **Control Server**: t2.micro (Ansible control node)
- **Node Server**: t2.medium (Monitoring stack host)

Security groups should allow inbound traffic on:
- TCP 22 (SSH)
- TCP 3000 (Grafana)
- TCP 9090 (Prometheus)
- TCP 9093 (Alertmanager)
- TCP 9100 (Node Exporter)

### 2. System Configuration

**On Control Server:**
```bash
sudo su -
hostnamectl set-hostname control
amazon-linux-extras install ansible2 -y
ansible --version
```

**On Node Server:**
```bash
sudo hostnamectl set-hostname node
```

### 3. SSH Key Setup

**Generate key pair on Control Server:**
```bash
cd /root/.ssh/
ls -ltr
ssh-keygen    # Press enter for defaults
cat id_rsa.pub
```

**Copy public key to Node Server:**
```bash
cd /root/.ssh/
vi authorized_keys   # Paste the copied key, save & exit (:wq)
chmod 600 authorized_keys
cd ..
chmod 700 .ssh
```
Now test connection from Control Server:
```bash
ssh <NODE_PUBLIC_IP>
```

Test Ansible Connectivity
```bash
On Control Server:

ansible all -m ping
You should see "pong" as output
```

### 4. Ansible Configuration

**Clone the repository:**
```bash
cd /home/ec2-user
git clone https://github.com/Lubhitdevops/Ansible-Monitoring-Project.git
cd Ansible-Monitoring-Project
```

**Configure inventory file:**
```ini
[prometheus]
<node-private-ip> ansible_ssh_private_key_file=~/.ssh/ansible-monitoring
else
<node-public-ip>

[grafana]
<node-private-ip> ansible_ssh_private_key_file=~/.ssh/ansible-monitoring
else
<node-public-ip>

[alertmanager]
<node-private-ip> ansible_ssh_private_key_file=~/.ssh/ansible-monitoring
else
<node-public-ip>

[node_exporter]
<node-private-ip> ansible_ssh_private_key_file=~/.ssh/ansible-monitoring
else
<node-public-ip>

[monitoring:children]
prometheus
grafana
alertmanager
node_exporter
```

### 5. Deployment

**Execute the playbook:**
```bash
ansible-playbook -i inventory playbook.yml
```
**playbook.yml**
```bash
playbook.yml:

- hosts: alertmanager
  become: yes
  become_user: root
  become_method: sudo
  roles:
    - alertmanager

- hosts: node_exporter
  become: yes
  become_user: root
  become_method: sudo
  roles:
    - prometheus_node_exporter

- hosts: prometheus
  become: yes
  become_user: root
  become_method: sudo
  roles:
    - prometheus

- hosts: grafana
  become: yes
  become_user: root
  become_method: sudo
  roles:
    - grafana
```

## ✅ Verification

**Check service status on Node Server:**
```bash
systemctl status prometheus
systemctl status grafana-server
systemctl status alertmanager
systemctl status node_exporter
```

**Access the services on browser:**
- Prometheus: http://<NODE_PUBLIC_IP>:9090
- Grafana: http://<NODE_PUBLIC_IP>:3000 (admin/admin)
- Alertmanager: http://<NODE_PUBLIC_IP>:9093
- Node Exporter: http://<NODE_PUBLIC_IP>:9100/metrics

## 🔔 Alert Integration

The solution supports alert routing to multiple channels:

### Slack Integration
Edit `roles/alertmanager/templates/alertmanager.yml.j2` to include:
```yaml
receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#monitoring-alerts'
        api_url: 'https://hooks.slack.com/services/XXXXX/XXXXX/XXXXX'
```

### PagerDuty Integration
```yaml
receivers:
  - name: 'pagerduty-notifications'
    pagerduty_configs:
      - service_key: 'your-pagerduty-integration-key'
```

## 🛠️ Troubleshooting

**Common issues and solutions:**

1. **Connection refused errors**: Verify security group rules and service status
2. **Permission denied (publickey)**: Verify SSH key configuration and permissions
3. **Service start failures**: Check journalctl logs for specific services
   ```bash
   journalctl -u prometheus -f
   ```

## 📊 Next Steps

- Configure additional exporters (MySQL, Nginx, Apache)
- Set up TLS/SSL encryption for web interfaces
- Implement backup strategies for Prometheus data
- Configure Grafana data sources and dashboards
- Set up alert rules in Prometheus

## 📜 License

This project is licensed under the MIT License. See LICENSE file for details.

## 🙏 Acknowledgments

- **Ansible** for infrastructure automation
- **Prometheus** for metrics collection and monitoring
- **Grafana** for visualization capabilities
- **Alertmanager** for alert management
- **Node Exporter** for system metrics collection

---
