# Node-App-Deployment-with-Ansible

![image](https://github.com/kenchuks44/Node-App-Deployment-with-Ansible/assets/88329191/1284b067-1993-4390-b679-c1b145f8ff3f)

## What is Ansible
Ansible is an open-source automation tool that simplifies configuration management, application deployment, and task automation through agentless, declarative playbooks written in YAML.

## What are ARM Templates
ARM templates are JSON files that assist in implementing an Azure infrastructure-as-code solution, allowing for automated and repeatable provisioning of resources

## Objective
We will be using ARM template and Ansible to automate a Node App deployment 

## Project Steps
# Step 1: Install Ansible using the codes below:
```
sudo pip install ansible
```

# Step 2: Confirm Ansible installation using the code below
```
ansible --version
```

![Screenshot (513)](https://github.com/kenchuks44/Node-App-Deployment-with-Ansible/assets/88329191/9a8f9e8a-b414-4e4b-b296-1729957bacb2)

# Step 3: Create ssh keys on Azure portal

![Screenshot (531)](https://github.com/kenchuks44/Node-App-Deployment-with-Ansible/assets/88329191/7ec723a2-8d32-4364-94f6-62974d3c8fc1)

# Step 4: Deploy a VM using ARM template

![Screenshot (529)](https://github.com/kenchuks44/Node-App-Deployment-with-Ansible/assets/88329191/dc724684-91f0-49e5-ba3e-8575a1c4a68e)

![Screenshot (524)](https://github.com/kenchuks44/Node-App-Deployment-with-Ansible/assets/88329191/8b9114fc-ef4f-4d5e-bc92-bc965c224324)

![Screenshot (525)](https://github.com/kenchuks44/Node-App-Deployment-with-Ansible/assets/88329191/52a06c41-e317-42f3-81c1-3154e7c8b256)

![Screenshot (526)](https://github.com/kenchuks44/Node-App-Deployment-with-Ansible/assets/88329191/49d56785-2604-4906-9d6d-b8fc0e5ce331)

![Screenshot (538)](https://github.com/kenchuks44/Node-App-Deployment-with-Ansible/assets/88329191/a3f343fe-77ff-4869-a9b9-d2edfcbf31b6)

## What is Ansible Inventory File
In Ansible, an inventory file is a text file that contains the list of hosts(remote servers) that Ansible can manage. The inventory file is used to define and organize the hosts into groups and assign variables to them. Ansible uses this to determine the target hosts for running plays and tasks.

## Step 5: Create an Ansible Inventory File using the code below:
```
vi hosts.txt
```
Then insert the code below:
```
[target_server]
<remote-server-ip> ansible_ssh_private_key_file=/<path-to-ssh-key/ ansible_user=azureuser
```
## What is an Ansible Configuration File
An Ansible configuration file, often named ansible.cfg, allows users to centrally define default settings, such as the inventory file path and SSH configuration, for maintaining consistency across Ansible playbooks and projects.

## Step 6: Create an Ansible Configuration File in the Project home directory using the codes below:
```
vi ansible.cfg
```
Then insert the code below:
```
[defaults]
host_key_checking = False
inventory = hosts
```

## What are Ansible Variables
In Ansible, variables are used to store and manipulate data within playbooks, roles, and tasks. They allow us to parameterize our configuration and make it more dynamic

## Step 7: Create Ansible Variables file using the codes below:
```
vi project-vars
```
Then insert the code below:
```
version: 1.0.0
location: /home/ken/ansible
linux_name: ken
user_home_dir: /home/{{linux_name}}
```

## What are Ansible Modules
Ansible modules are reusable, standalone scripts that perform specific tasks on remote hosts, allowing automation of various operations such as system administration, configuration management, and application deployment.

Here are a few examples of Ansible modules in action:

Shell Module: 
- Execute a shell command on remote hosts.
- ansible -m shell -a "ls -l" target_server

User Module: 
- Create a user on remote hosts.
- ansible -m user -a "name=johndoe password=<encrypted_password>" target_server

Copy Module: 
- Copy a file from the local machine to remote hosts.
- ansible -m copy -a "src=/path/to/local/file.txt dest=/remote/path/" webservers

File Module: 
- Ensure a file exists or absent on remote hosts.
- ansible -m file -a "path=/path/to/file state=present" target_server

Service Module: 
- Ensure a service is running on remote hosts.
- ansible -m service -a "name=apache2 state=started" target_server

Package Module: 
- Ensure a package is installed on remote hosts.
- ansible -m package -a "name=nginx state=present" target_server

## What is an Ansible Playbook
An Ansible playbook is a YAML-formatted file that defines a set of tasks to be executed on remote hosts in an infrastructure automation scenario. Playbooks are a fundamental component of Ansible, an open-source automation tool. Each playbook consists of one or more plays, where each play is a set of tasks executed on a specific group of hosts. Tasks, written in YAML syntax, define the desired state of the system by specifying actions such as installing packages, configuring settings, or performing other operations.

## Create an Ansible playbook to deploy Node App using the codes below:
```
vi node-deploy.yaml
```
Then insert the codes below:
```
---
- name: Install node and npm
  become: yes
  hosts: target_server
  tasks:
    - name: Update apt repo and cache
      apt: update_cache=yes force_apt_get=yes cache_valid_time=3600
    - name: Install nodejs and npm
      apt:
        pkg:
          - nodejs
          - npm
          - acl

- name: Create new linux user for node app
  become: yes
  hosts: target_server
  tasks:
    - name: Create user
      user:
        name: ken
        comment: Ken Admin
        group: admin

- name: Deploy nodejs app
  hosts: target_server
  become: yes
  become_user: ken
  vars_files:
    - project-vars
  tasks:
    - name: Copy nodejs file
      ansible.builtin.copy:
        src: "/home/ken/simple-nodejs-master.tar"
        dest: "{{ user_home_dir }}"
    - name: Unpack nodejs tar file
      ansible.builtin.command: tar -xf "{{ user_home_dir }}/simple-nodejs-master.tar" -C /home/ken
    - name: Install dependencies
      npm:
        path: /home/ken/package
    - name: Start the application
      command: 
        chdir: "{{user_home_dir}}/package/app"
        cmd: node server
      async: 1000
      poll: 0  
    - name: Ensure app is running
      shell: ps aux | grep node
      register: app_status
    - debug: msg={{app_status.stdout_lines}}
```

Execute the playbook using the command below:
```
ansible-playbook -i hosts node-deploy.yaml
```

![Screenshot (535)](https://github.com/kenchuks44/Node-App-Deployment-with-Ansible/assets/88329191/18a498f4-7035-45e2-a27c-8b0e04a56516)

![Screenshot (536)](https://github.com/kenchuks44/Node-App-Deployment-with-Ansible/assets/88329191/07537a57-9e21-44cf-80fd-c48b26c5b602)

![Screenshot (539)](https://github.com/kenchuks44/Node-App-Deployment-with-Ansible/assets/88329191/1af7b4bf-71c6-474d-b727-1d5d8f9a57e2)

## Congratulations








