#Install Ansible on RHEL (Controller Node)
sudo yum update -y
sudo yum install epel-release -y
sudo yum install ansible -y
ansible --version
ssh-keygen -t rsa -b 4096
ssh-copy-id ec2-user@<amazon-linux-node-ip>
ssh ec2-user@<amazon-linux-node-ip>

# Set Up Ansible Inventory
- Edit the inventory file (default: /etc/ansible/hosts):
[amazon_linux]
node1 ansible_host=<ip1> ansible_user=ec2-user
node2 ansible_host=<ip2> ansible_user=ec2-user
node3 ansible_host=<ip3> ansible_user=ec2-user
- Test connectivity:
ansible -m ping amazon_linux

# ⚙️ Step 4: Run a Simple Command:
ansible amazon_linux -m shell -a "uname -a"

# Create a Playbook File
install_apache.yml:
- name: Install and start Apache on Amazon Linux nodes
  hosts: amazon_linux
  become: yes

  tasks:
    - name: Ensure Apache (httpd) is installed
      yum:
        name: httpd
        state: present

    - name: Ensure Apache service is enabled and running
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Deploy custom index.html
      copy:
        dest: /var/www/html/index.html
        content: "<h1>Hello from Ansible!</h1>"


#⚙️ Run the Playbook
Execute the playbook from your controller:
ansible-playbook install_apache.yml


# Step 3: Verify Installation
- Check service status on one of the nodes:
systemctl status httpd
- Open a browser and visit:
http://<amazon-linux-node-ip>


# Step 1: Create the Role Skeleton
Run:
ansible-galaxy init apache


# Step 2: Define Tasks
Edit roles/apache/tasks/main.yml:
---
- name: Ensure Apache (httpd) is installed
  yum:
    name: httpd
    state: present

- name: Deploy custom index.html
  copy:
    dest: /var/www/html/index.html
    content: "<h1>Hello from Ansible Role!</h1>"

- name: Ensure Apache service is enabled and running
  service:
    name: httpd
    state: started
    enabled: yes
  notify: Restart Apache


#Step 3: Add a Handler
Edit roles/apache/handlers/main.yml:
---
- name: Restart Apache
  service:
    name: httpd
    state: restarted


# Step 4: Create a Playbook That Uses the Role
Create site.yml:
---
- name: Configure web servers
  hosts: amazon_linux
  become: yes
  roles:
    - apache


#Step 5: Run the Role
Execute:
ansible-playbook site.yml


# Step 1: Add Variables
Create a file roles/apache/vars/main.yml:
---
apache_port: 80
apache_index_content: "<h1>Hello from {{ inventory_hostname }}!</h1>"


# Step 2: Create a Template
Create roles/apache/templates/index.html.j2:
<html>
  <head><title>Welcome</title></head>
  <body>
    {{ apache_index_content }}
  </body>
</html>


# Step 3: Update Tasks to Use Variables & Template
Edit roles/apache/tasks/main.yml:
---
- name: Ensure Apache (httpd) is installed
  yum:
    name: httpd
    state: present

- name: Deploy custom index.html from template
  template:
    src: index.html.j2
    dest: /var/www/html/index.html

- name: Ensure Apache service is enabled and running
  service:
    name: httpd
    state: started
    enabled: yes
  notify: Restart Apache


# Step 4: Handler (unchanged)
roles/apache/handlers/main.yml:
---
- name: Restart Apache
  service:
    name: httpd
    state: restarted


# Step 5: Run the Playbook
Execute:
ansible-playbook site.yml




