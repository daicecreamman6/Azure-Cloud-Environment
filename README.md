## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![alt text](https://github.com/daicecreamman6/Azure-Cloud-Environment/blob/main/Diagrams/Azure%20cloud%20environment.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the YML files below may be used to install only certain pieces of it, such as Filebeat.

```yaml
---
- name: Configure Web VM with Docker
  hosts: webserver
  become: true
  tasks:
  - name: Install docker.io
    apt:
       update_cache: yes
       name: docker.io
       state: present
  - name: Install pip3
    apt:
      name: python3-pip
      state: present
  - name: Install Docker python module
    pip:
      name: docker
      state: present
  - name: Download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      restart_policy: always
      published_ports: 80:80
  - name: Enable Docker service
    systemd:
      name: docker
      enabled: yes
  ```
```yaml
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: azureuser
  become: true
  tasks:
  # Use apt module
  - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

  # Use apt module
  - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

  # Use pip module (It will default to pip3)
  - name: Install Docker module
      pip:
        name: docker
        state: present

  # Use command module
  - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

  # Use sysctl module
  - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

  # Use docker_container module
  - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          -  5601:5601
          -  9200:9200
          -  5044:5044

  # Use systemd module
  - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```        
```yaml
---
- name: installing and launching filebeat and metricbeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start

  - name: enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes
      
#install metricbeat

  - name: download metricbeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

  - name: install metricbeat deb
    command: dpkg -i metricbeat-7.4.0-amd64.deb

  - name: drop in metricbeat.yml
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

  - name: enable and configure docker module for metricbeat
    command: metricbeat modules enable docker

  - name: setup metricbeat
    command: metricbeat setup

  - name: start metricbeat service
    command: service metricbeat start

  - name: enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes
  ```
  

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network. Load balancers protect the availability of the servers by distributing network traffic to the web servers behind it. This helps minimize the effects of a DDOS attack. Our Jumpbox acts as a gateway to our configuration environment, letting us configure multiple virtual machines easily In one location.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the data and system logs. Filebeat monitors the system files for any changes while metric beat monitors the system and service information.

The configuration details of each machine may be found below.

| Name     | Function | IP Address | Operating System    |
|----------|----------|------------|------------------   |
| JumpBox  | Gateway  | 10.0.0.4   | Linux (Ubuntu 20.04)|
| WEB-1    |Web server| 10.0.0.5   | Linux (Ubuntu 20.04)|
| WEB-2    |Web server| 10.0.0.6   | Linux (Ubuntu 20.04)|
| ELKVM    |Log server| 10.1.0.4   | Linux (Ubuntu 20.04)|

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jumpbox provisioner (IP 10.0.0.4) can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
(My home IP) on port 22

Machines within the network can only be accessed by Jumpbox.
I only allowed my Jumpbox VM to access my ELKvm via port 22. Pcs with the ip address of (My home IP) on port 5602 can access the ELKvm.

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| JumpBox  | Yes                 | (My Home IP)         |
| WEB-1    | no                  | 10.0.0.5             |
| WEB-2    | no                  | 10.0.0.6             |
| ELKvm    | no                  | (My home IP):5602    |
### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because they use configuration as code to easily re-deploy identical virtual machines quickly and efficiently.

The playbook implements the following tasks:
- Installs docker, PIP3, and python module
- Increse VM memory allocation use: sysctl -w vm.max_map_count =262144

- Download and install Elk container and maps port 5601 to 5601 and 9200 t0 9200

![The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance](Diagrams/Docker ps.PNG)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
WEB-1 – 10.0.0.5
WEB-2 – 10.0.0.6

We have installed the following Beats on these machines:
 Filebeat 7.4.0
 Metricbeat 7.4.0

These Beats allow us to collect the following information from each machine:
Filebeat monitors the log files or specified locations, collects log events and forwards them to Elasticsearch or logstash for indexing.
Metricbeat collects system-level logs, such as CPU usage, memory, file systems, disk IO, and network IO statistics as well as all running proccesses

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Within the ansible container,
- Update the hosts file to indicate which machine to install the elk server on.
 - Update the filebeat-config.yml file to include the following: 
 - username and password for ELKvm
 - private IP “10.1.0.4” of the Elk-Server to the ElasticSearch and Kibana sections of the configuration file
- Run the playbook Filebeat-playbook.yml which will make a copy of the playbook in /etc/ansible/roles directory of the ELKvm and will copy the filebeat-config.yml to /etc/ansible/files and
  Navigate to http://[0.0.0.0]:5601/app/kibana to verify that it works.

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files,
To download files from GitHub, you navigate to the top level of the project (Azure-Cloud-Environment) Look for the green "Code" download button on the right. Choose the Download ZIP option from the Code pull-down menu.
