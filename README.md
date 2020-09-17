# Elk-PROJECT

## Automated ELK Stack Deployment

This document contains the following details:

*	Description of the Topology
*	ELK Configuration
	1.	Beats in Use
	2.	Machines Being Monitored
*	How to Use the Ansible Build 
*	Access Policies


The files in this repository were used to configure the network depicted below.

![alt text](https://github.com/trippyville/PROJECT-ELK/blob/master/Diagrams/HW12%20Diagram-Page-1.jpg)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the ansible-playbook file may be used to install only certain pieces of it, such as Filebeat.

CLICK HERE for ![Ansible-Playbook File link]( https://github.com/trippyville/PROJECT-ELK/tree/master/Ansible)

### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.
Load balancing ensures that the application will be highly AVAILABLE, in addition to restricting INBOUND ACCESS to the network. The Load Balancers will help SSL offload and traffic compression. Load balancer also protect applications from emerging threats, DDoS attacks, and authenticate user accounts.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the FILE SYSTEM OF THE VMs on the NETWORK and system METRICS. 
	
*	FILEBEAT monitors the log files or locations that is specify, collects log, events, and forwards them to ELK. (source :Elastic.co)
*	METRICBEAT helps monitor servers by collecting metrics from the system and services running on the server, such as APACHE. (source: Elastic.co)

The configuration details of each machine may be found below. 

| Name              | Function   | IP Address   | Operation System |
|-------------------|------------|--------------|------------------|
| JumpBox-Provision | Geteway    | 52.183.90.25 | Linux            |
| REDteam-VM-DVWA   | Web-Server | 10.10.0.8    | Linux            |
| BLUEteam-VM-DVWA2 | Web-Server | 10.10.0.11   | Linux            |
| ELK               | Monitoring | 40.74.246.77 | Linux            |

## Access Policies

The machines on the internal network are not exposed to the public Internet.
Only the JUMPBOX machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses: 13.77.144.49 (root@90423f24817)
Machines within the network can only be accessed by EACH OTHER. – only the JUMPBOX-provision to docker is able access ELK. Container ID 490423f24817
A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses                         |
|----------|---------------------|----------------------------------------------|
| Jump Box | YES                 | Public IP 52.183.90.25 / private IP 10.10.0.7|
| ELK      | NO                  | Public IP 40.74.246.77 / private IP 10.0.0.4 |
| REDteam  | NO                  | 10.10.0.8                                    |
| BLUEteam | NO                  | 10.10.0.11                                   |

## Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because it can set up various servers you need in your infrustracure like database, storage devices, networks, firewalls, etc.  (source: cloudacademy)

The playbook implements the following tasks: 
- 
	Scp -i id_rsa  install_elk.yml root@490423f24817:/root 
or create a file inside your root container
	
	nano install_elk.yml 
	
copy and paste yml file with the file link [HERE](https://github.com/trippyville/PROJECT-ELK/blob/master/Ansible/install-elk.yml).

Then make sure to update your docker by using these command:

	apt update
	apt upgrade
	ansible-playbook install_elk.yml

-*ALERT – make sure to provide ELK VM with enough memory to run-

Then ssh into your docker ELK VM with "ssh id_rsa azureuser@40.74.246.77"
Once inside the ELK VM, check if the ELK container is avaliable by using the command:
	
	sudo docker container list -a

If you see ELK in the container, go ahead and start the ELK by using this command:

	sudo docker start elk
	
Check is the ELK container is running:
	
	sudo docker ps
	
If it appears with the "sudo docker ps command", it is now active! Head on over to your browser and type in (your-ELK-IPaddress):5601 
*NOTE: you may have to wait a few minutes before it shows up on your browser. 

The following screenshot displays the result of running docker ps after successfully configuring the ELK instance.

![Sudo docker ps reference](https://github.com/trippyville/PROJECT-ELK/blob/master/Reference/sudo%20docker%20start%20elk%20ps.PNG) 

	##install ELK
	---
  	- name: install elk
   	  hosts: elk
    	  become: true
          tasks:
    	- name: docker.io
          apt:
          name: docker.io
          state: present
        
    	- name: Increase virtual memory
          command: sysctl -w vm.max_map_count=262144

    	- name: Install pip3
          apt:
          name: python3-pip
          state: present

    	- name: Install Docker python module
          pip:
          name: docker
          state: present

    	- name: download and launch a docker web container
          docker_container:
          name: elk
          image: sebp/elk:761
        - name: download and launch a docker web container
          state: started
          published_ports:
           - "5601:5601"
           - "9200:9200"
           - "5044:5044"

Target Machines & Beats
This ELK server is configured to monitor the following machines: REDteam & BLUEteam VMs, at 10.10.0.8 and 10.10.0.11.
We have installed the following Beats on these machines: 

*	Filebeat
*	MetricBeat (not used during our course)
*	Packbeat
  
These Beats allow us to collect the following information from each machine: 

*	Filebeat: Detects changes to the filesystem like Apache logs
*	Metricbeat: Detects changes in system metrics, such as CPU usage, detect SSH login attempts, failed sudo escalations, and CPU/RAM statistics. 
*	Packetbeat: Collects packets that pass through the NIC, similar to Wireshark. IT generate trace of all activity through the network for forensic analysis. 

The playbook below installs Filebeat on the target hosts. The playbook for installing Metricbeat is not included, but looks essentially identical — simply replace Filebeat with Metricbeat  and it will work as expected. NOTE: curl the update http link*

	##installing Filebeat
	---
		- name: Installing and Launch Filebeat
		  hosts: webservers
  		  become: yes
  		  asks:
   	 #Use command module
  		- name: Download filebeat .deb file
  		  command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb
    
   	 #Use command module
 		- name: Install filebeat .deb
    		  command: sudo dpkg -i filebeat-7.6.1-amd64.deb

    	#Use copy module
  		- name: Drop in filebeat.yml
    		  copy:
      		  src: /etc/ansible/files/filebeat-config.yml
      		  dest: /etc/filebeat/filebeat.yml

   	#Use command module
  		- name: Enable and Configure System Module
    		  command: filebeat modules enable system

    	#Use command module
  		- name: Setup filebeat
    	          command: filebeat setup

    	#Use command module
  		- name: Start filebeat service
   		  command: service filebeat start

## Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned we will use JUMPBOX for this purpose.
 
	$ cd /etc/ansible
	$ mkdir files 
	
	# Clone Repository + IaC Files
	
The easiest way is to cp the config or yml file or touch nano a new .config or yml file into the docker container in their respecitive folder.

Next, you must create a hosts file to specify which VMs to run each playbook on. 

	Run this command to insert syntax below:

		$ cd /etc/ansible
		$ cat > hosts <<EOF

		 [webservers]
		10.10.0.8 ansible_python_interpreter=/usr/bin/python3
		10.10.0.11 ansible_python_interpreter=/usr/bin/python3
 
 		 [elk]
		10.0.0.4 ansible_python_interpreter=/usr/bin/python3

	EOF

After this, create a folder called “file” to store the file-beat config to run ansible.

Command as follow:

		Mkdir files /etc/ansible
 
 the commands below run the playbook:

	bash  $ cd /etc/ansible  
		$ ansible-playbook install_elk.yml elk
		$ ansible-playbook install_filebeat.yml webservers
		$ ansible-playbook install_metricbeat.yml webservers

To verify success, wait five minutes to give ELK time to start up.

Then, run: curl http://40.74.246.77:5601. This is the address of Kibana.
If the installation succeeded, this command should print HTML to the console.
