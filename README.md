Ansible Monitoring Project

This project sets up a complete monitoring stack using Ansible on AWS EC2 instances.
It installs and configures the following tools:

Prometheus – Metrics collection and monitoring

Grafana – Visualization and dashboards

Alertmanager – Sending alerts to PagerDuty / Slack

Node Exporter – Exporting server-level metrics

All tools are configured as systemd services so they can be easily started, stopped, and managed.

🚀 Technologies Used

Amazon EC2 (Amazon Linux 2, but works with other AMIs too)

Ansible (Control node for automation)

Prometheus, Grafana, Alertmanager, Node Exporter

PagerDuty / Slack integration for alerts

📋 Project Architecture

Control Server (EC2):

Runs Ansible to automate setup on the Node server.

Configured with SSH key-based authentication.

Node Server (EC2):

Installs and runs Prometheus, Grafana, Alertmanager, Node Exporter as services.

⚙️ Setup Steps
1. Launch EC2 Instances

Create 2 EC2 instances:

control → Ansible Control Server

node → Monitoring Node Server

Allow All TCP (Custom IPv4 Anywhere) inbound rule on the Control Server for connectivity.

2. Install Ansible on Control Server
sudo su -
amazon-linux-extras install ansible2 -y
ansible --version

3. Set Hostnames

On Control Server:

hostnamectl set-hostname control


On Node Server:

hostnamectl set-hostname node

4. Setup SSH Key Authentication

On Control Server:

cd /root/.ssh/
ls -ltr
ssh-keygen    # Press enter for defaults
cat id_rsa.pub


Copy the key.

On Node Server:

cd /root/.ssh/
vi authorized_keys   # Paste the copied key, save & exit (:wq)
chmod 600 authorized_keys
cd ..
chmod 700 .ssh


Now test connection from Control Server:

ssh <NODE_PUBLIC_IP>

5. Test Ansible Connectivity

On Control Server:

ansible all -m ping


You should see "pong" as output.

6. Clone Repository

On Control Server:

cd /home/ec2-user
git clone https://github.com/Lubhitdevops/Ansible-Monitoring-Project.git

7. Configure Inventory

Edit the inventory file and add the Node Server public IP under the respective groups:

[prometheus]
<node-public-ip>

[grafana]
<node-public-ip>

[alertmanager]
<node-public-ip>

[node_exporter]
<node-public-ip>

8. Run Playbook

Execute the playbook:

ansible-playbook -i inventory playbook.yml


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

✅ Verification
1. Check Services on Node Server
systemctl status prometheus
systemctl status grafana-server
systemctl status alertmanager
systemctl status node_exporter


All should show active (running).

2. Access Tools in Browser

Prometheus → http://<NODE_PUBLIC_IP>:9090

Grafana → http://<NODE_PUBLIC_IP>:3000 (Default login: admin/admin)

Alertmanager → http://<NODE_PUBLIC_IP>:9093

Node Exporter → http://<NODE_PUBLIC_IP>:9100/metrics

📡 Alerts Integration

Alertmanager can be configured to send alerts to Slack or PagerDuty by editing the alertmanager.yml file under the role.

📜 License

This project is licensed under the MIT License – you are free to use, modify, and distribute it.

🙏 Acknowledgements

Ansible

Prometheus

Grafana

Alertmanager

Node Exporter
