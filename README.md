## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![](https://github.com/imntru/Elk-Stack--Project/blob/main/Diagrams/Cloud%20Security%20Homework%20.jpg)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the YML file may be used to install only certain pieces of it, such as Filebeat.

  - [DVWA/Docker Install](https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/DVWA.yml)
  - [Elk Install](https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/install-elk.yml)
  - [Filebeat](https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/filebeat-install.yml)
  - [Metricbeat install](https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/metric-beat.yml)

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
- <i>Load balancers</i> aid in securing a network by providing continuous availability through distribution of incoming traffic to web servers, this ensures that no, one server becomes overloaded as is common in DDoS (Denial of Service) attacks. A <i>Jump Box</i> aids in securing the network further by controlling access and administration of multiple systems by disallowing access without use of the private IPs of the machines. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the system logs and system metrics.
- <i> Filebeat </i> monitors the log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.
- <i> Metricbeat</i> helps monitor your servers by collecting metrics from the system and services running on the server.

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.4   | Linux (ubuntu 20.04) |
| Web 1    | Server   | 10.0.0.5   | Linux (ubuntu 20.04) |
| Web 2    | Server   | 10.0.0.6   | Linux (ubuntu 20.04) |
| Elk Server   | Log Server  | 10.1.0.4   | Linux (ubuntu 20.04) |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- Local Hosts IP, in my case: 73.12.59.111

Machines within the network can only be accessed by the Jump Box.
- The Elk server can connect to the local host over port 5601- IP: 73.12.59.111

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes/No              | 73.12.59.111         |
| Web 1    | No                  | 10.0.0.4             |
| Web 2    | No                  | 10.0.0.4             |
| Elk      | Yes (port 5601)     | 73.12.59.111         |
| Load Balancer | Yes (port 80)  | 73.12.59.111         |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because the services are running within a lightweight environment so system installation and update can be streamlined, and the processes can be more replicable due to the limited resources actually being utilized in deployment. 

The playbook implements the following tasks:

Installs docker.io, pip3 and the docker module:
```bash
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
```
Increases the virtual memory for the Elk Server to operate:
```bash
      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144
```
Ensures that the above setting is run automatically upon restart:
```bash
      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: sysctl
        value: '1'
        state: present
        reload: no
```
Downloads, installs and executes the docker Elk container on upon restart configured with the listed port mappings:
```bash
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
```
Enables docker service upon boot:
```bash
      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![Docker Output](https://github.com/imntru/Elk-Stack--Project/blob/main/Images/screen%20shot.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web 1 (10.0.0.5)
- Web 2 (10.0.0.6)

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat 

These Beats allow us to collect the following information from each machine:
- <i>Filebeat</i>- a log data collector that allows the recording of logs and their evolution through changes if they occur functions on the servers by monitoring log directories and files for changes and forwarding them to Elastisearch or Logstash for indexing. For example by usage of filebeat you can specify log types within the Logs UI to track activity across aggregated logs based on categories such as apps, hosts, services or data centers. 
- <i> Metricbeat</i> allows us to collect metrics on the system-level CPU usage, memory, file system, disk IO, and network IO statistics, as well as top-like statistics for every process running on your systems. For example Metricbeat comes with internal modules that collect metrics from services like Apache, Jolokia, NGINX, MongoDB, MySQL, PostgreSQL, Prometheus, and more.


### Using the Playbook
In order to use the playbook, you
will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the configuration file from your Ansible container to Web 1 and Web 2.
- Update the /etc/ansible/hosts file to include the IP addresses of your Elk server, and Web servers.
- Run the playbook, and navigate to http://[Elk_VM_Public_IP] to check that the installation worked as expected.

Which file is the playbook? 
- [filebeat-install.yml](https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/filebeat-install.yml)

Where do you copy it?
- /etc/ansible/filebeat-config.yml to /etc/filebeat/filebeat.yml

Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?
-In order to specify the specific machine to install the Elk server on make sure the [install-elk.yml](https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/install-elk.yml) indicates the machine under:
```bash
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: azdmin
  become: true
  tasks:
```
*NOTE* Make sure the name under host: is the same within the host file: for instance, within this project specificaly three machines were created:
2 Webservers
1 Elk Server
- these are indicated clearly within the hosts file 
``` bash
[webservers]
10.0.0.5
10.0.0.6

[elk]
10.1.0.4
```
In order to specify which machines to run filenbeat on update the [filebeat-install.yml](https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/filebeat-install.yml) and as state above make sure to indicate the group on machines in which to install filebeat:
```bash
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:
```
*Note*: Make sure to indicate the output for filebeat is specified within the [filebeat-config.yml](https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/filebeat-config.yml), the machines public IP address should be placed under the Elasticearch output header, under
```bash
  hosts: ["10.1.0.4:9200"]
  username: "elastic"
  password: "changeme" # TODO: Change this to the password you set
```
and under the Kibana header, under:
```bash
setup.kibana:
  host: "10.1.0.4:5601"
```


Which URL do you navigate to in order to check that the ELK server is running?
- http://[your.ELK-VM.External.IP]:5601/app/kibana

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._

Using ``` curl ``` to avoid errors you can directly download the configuration files for filebeat and metric beat:

- Run: ```curl https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/filebeat-config.yml > /etc/ansible/filebeat-config.yml```
for filebeat
- Run:
```curl https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/metricbeat%20config.yml > /etc/ansible/metricbeat-config.yml```
for metric beat.

- To decrease the chances of corruption of the information for the instllation files I would recommend downloading the yaml files within the github repository directly to your local machine and copying them out of Visual Studio to maintain the integrity of the files.
- [elk-install](https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/install-elk.yml)
- [filebeat-install](https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/filebeat-install.yml)
- [metricbeat-install](https://github.com/imntru/Elk-Stack--Project/blob/main/Ansible/metric-beat.yml)

-Copy these files playbooks into nano or vim files within the /etc/ansible/ directory by using this command:

``` sudo nano ____-playbook.yml```

using ansible you will be able to run these by running:

```ansible-playbook ____-playbook.yml```
