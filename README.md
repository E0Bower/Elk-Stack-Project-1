# Elk-Stack-Project-1
This Repository contains my submission for the Elk-Stack project. 
This document contains the following: 
  - Network Diagram 
  - Description of Deployment
  - Access Policies and Network Addresses 
  - Kibana Investigation
  - Usage Instructions

**Network Diagram**

  ![Project 1 Network Diagram](https://user-images.githubusercontent.com/89867644/146123251-d45ec257-edc9-4a2a-897e-5dd6d1695fab.png)

**Description of Deployment**

This virtual network was created to expose a monitored DVWA. A load balancer was deployed to ensure the availability of the DVWA while restricting unwanted web access to the virtual network.

The Red Team resource group consists of the following resources: 
- Red Team Virtual Network 
- Red Team Security Group 
- Red Team Availability Set
- Jump Box Provisioner Gateway
- Web-1 Webserver
- Web-2 Webserver
- Web-3 Webserver
- Red Team Load Balancer and Health Probe
- Elk Virtual Network 
- Elk Security Group
- Elk Webserver 

A jump box provisioner served as a launching point for an ansible container and provided access to all virtual machines within the Red Team Resource group.
Jump Box allows for secure administrative task completion and houses the ansible container, configuration files, and playbooks for services such as Filebeat and Metricbeat. 

Three additional virtual machines were created to house the DVWA. Access to these machines was restricted to an IP address and an encrypted SSH key.
Web-1,2, and 3 house docker containers and must be running in order to access the DVWA via the IP address of the Red Team Load Balancer.  

The Elk Virtual Network was deployed with an Elk Webserver that would allow access to Kibana. A two-way peering connection was established between Elk and Red Teams virtual networks. This peering allowed for access to the Elk webserver through the ansible container within the jump box. 

**Filebeat**

Filebeat was deployed to the ansible container to collect data regarding the file system of each machine. 

**Metricbeat**

Metricbeat was deployed to the ansible container to collect the metric data of each machine. 

**Kibana**

Kibana collects metrics and log analytics from the deployed web server for visualization. Kibana is only accessible via the IP address of the Elk webserver. 

**Tables for Access Policies and network addresses**

The below table details the configurations of each deployed machine. 

![Configuration table](https://user-images.githubusercontent.com/89867644/146123740-2fa0612c-e8d7-42fd-ad39-42ff7878012b.PNG)

**The following table details the access policies of each deployed machine.** 

![Access Policy Table](https://user-images.githubusercontent.com/89867644/146123834-89847c1a-a7a2-4911-8a3a-f2e2df044232.PNG)

**Kibana Investigation** 

The following section contains my findings from a Kibana investigation of sample weblog data. 

**General Overview**

	Over the last 7 days (relative to 12/14/2021) There were 228 unique visitors located in India. In the last 24 hours, 9 visitors from China were using Mac OSX. In the last 2 days, approximately 35% of users received 404 errors while approximately 14% of users received 503 errors. In the last 7 days, users from China produced the majority of web traffic to the site. Web traffic from China was the highest around 1:00 PM. Over the last 7 days css, deb, gz, rpm and zip files were downloaded from the website. 

**Event Investigation**

	On December 12, 2021, the average bytes peaked at 20:55 (8:55 PM). There was one unique visitor at peak of this activity. Upon further investigation, it was found that this user downloaded an rpm file (red hat package manager). I found that the User sent an HTTP GET request and received a response code of 200. I determined that the activity originated in the United State according to the following geo coordinates: lat: 43.34121, lon: -73.6103075. The source IP address was 35.143.166.159 and the source machine was operating on Windows 8. 

  	The user accessed https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-6.3.2-i686.rpm and downloaded the rmp file. The user was apparently redirected from a Facebook page at the URL http://facebook.com/success/jay-c-buckey. The file downloaded appeared to be an rpm file used for metric data collection. Files like this are not inherently malicious but are still susceptible to exploitation. Users can download rpm files and manipulate web data. The event did not appear to be suspicious, however, the source IP address was located in the United States, but the source location was listed as India. Additionally, one user generated an abnormally large number of bytes. It is possible that due to a slow connection, the rpm file download may have been stopped and restarted multiple times. According to the timestamp, the event began at 20:55 and was not completed until 20:57. This could be because of a slow download or an unreliable TCP connection which would account the for the amount of data generated by a single user. It is possible that the Facebook page was hosted offshore, specifically from India. The user may have navigated to the Facebook page and clicked the download link which redirected them to the metricbeat rpm file.


**Usage Instructions**

To utilize [Filebeat](https://github.com/E0Bower/Elk-Stack-Project-1/blob/dc30b5251c2a0be3923a01c191b5a77046bf6dbc/Ansible/filebeat-playbook.yml) and [Metricbeat](https://github.com/E0Bower/Elk-Stack-Project-1/blob/dc30b5251c2a0be3923a01c191b5a77046bf6dbc/Ansible/metricbeat-playbook.yml) playbooks you will need to configure an ansible control node as well as set up configuration files and install elk to your ansible container.  
SSH into your jump box and attach your ansible container to proceed with configuration and installation. 

Navigate to you /etc/ansible directory and nano into the hosts file. 

  $ cd /etc/ansible
  
  $ nano hosts 
  
  comment out [webservers] and [elk]
  
  under [webservers] enter the internal IP's of your Web VMs followed by ansible_python_interpreter=user/bin/python3
  
  [webservers]
  
  10.0.0.8 ansible_python_interpreter=user/bin/python3
  
  10.0.0.9 ansible_python_interpreter=user/bin/python3
  
  under [elk] repeat the same steps with your Elk server's internal IP
  
 Navigate back to /etc/ansible and create install-elk.yml. This file will serve as your elk playbook. 
  
  $ touch install-elk.yml
  
  $ nano install-elk.yml
  
 A copy of an elk installation playbook is provided below. 
 
[Elk playbook](https://github.com/E0Bower/Elk-Stack-Project-1/blob/dc30b5251c2a0be3923a01c191b5a77046bf6dbc/Ansible/install-elk.yml)


Once your elk playbook has finished running, create the files and roles directories. these directories will house your filebeat and metricbeat configurations files and playbooks. 

  $ mkdir files
  
  $ mkdir roles 
 
navigate to the files folder to create your filebeat amd metricbeat configuration files. 

within these files you will need to add the inter IP of your elk server and allow connections to ports 9200 and 5601

**Filebeat configuration example:**

output.elasticsearch:

    hosts: ["10.1.0.4:9200"]

    username: "elastic"

    password: "changeme"

    setup.kibana:
  
    host: "10.1.0.4:5601"

**Metricbeat configuration example.**

output.elasticsearch:

    hosts: ["10.1.0.4:9200"]
  
     username: "elastic"
  
     password: "changeme"

setup.kibana:

    host: "10.1.0.4:5601"
    
Once you have completed these steps you are ready to create your playbooks. 

Once your playbooks for filebeat and metricbeat have run ensure that you can access Kibana and collect log data and docker metrics. 

http://[Elk-server-IP]:5601/app/kibana

