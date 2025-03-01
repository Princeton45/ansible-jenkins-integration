# Ansible Integration in Jenkins (CD)

![diagram](https://github.com/Princeton45/ansible-jenkins-integration/blob/main/images/diagram.png)

## Overview

This project demonstrates my implementation of Ansible integration with Jenkins for automated infrastructure configuration. I used Jenkins to orchestrate Ansible playbooks that configure AWS EC2 instances as part of a CD (Continous Deployment) pipeline.

### Pipeline Workflow

My Jenkinsfile implements the following workflow:

1. Connects to the remote Ansible Control Node server
2. Copies all Ansible playbooks and configuration files to the Control Node
3. Transfers the required SSH keys for the Managed Node servers to the Controller node so the Controller Node can authenticate to the EC2 instances.
4. Installs Ansible, Python3, and Boto3 on the Control Node if not already present
5. Executes the playbook remotely to configure the two EC2 instances (playbook installs Docker and Docker compose)

## Technologies Used

- Ansible
- Jenkins
- DigitalOcean
- AWS
- Boto3
- Docker
- Java
- Maven
- Linux
- Git

## Implementation Steps


### Server Setup

I created a dedicated server for Jenkins using DigitalOcean, installing and configuring it to handle our CD (Continuous Deployment) needs.

![jenkins](https://github.com/Princeton45/ansible-jenkins-integration/blob/main/images/jenkins.png)

For Ansible, I set up a separate control node server, also on DigitalOcean, which serves as the central point for running playbooks.

![ansible-server](https://github.com/Princeton45/ansible-jenkins-integration/blob/main/images/ansible-server.png)

I then created 2 EC2 instances that will be managed and configured by the Ansible server.

I also created a key pair for the instances so Ansible can use it to authenticate to them.

![ec2](https://github.com/Princeton45/ansible-jenkins-integration/blob/main/images/ec2.png)


### Ansible Configuration

I wrote an Ansible playbook that configures two EC2 instances with all necessary services and dependencies. The playbook handles installation, configuration, and service management.

The playbook also utilizes `ansible.cfg` for config settings and `inventory_aws_ec2.yaml` for dynamic inventory

`my-playbook.yaml`
```yaml
---
- name: Install Docker
  hosts: aws_ec2
  become: yes
  tasks:
    - name: Install Docker
      yum:
        name: docker 
        update_cache: yes
        state: present
    - name: Start docker daemon
      systemd:
        name: docker
        state: started

- name: Install Docker-compose
  hosts: aws_ec2
  tasks:
    - name: Create docker-compose directory
      file:
        path: ~/.docker/cli-plugins
        state: directory
    - name: Get architecture of remote machine
      shell: uname -m
      register: remote_arch
    - name: Install docker-compose
      get_url: 
        url: "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-{{ remote_arch.stdout }}"
        dest: ~/.docker/cli-plugins/docker-compose
        mode: +x
```
### Jenkins Integration

I added SSH key file credentials in Jenkins for both the Ansible Control Node server and the Ansible Managed Node servers, ensuring secure communication between all components.

The Jenkins pipeline was configured to execute the Ansible playbook on the remote Control Node as part of the CD process.


![Pipeline Execution](suggested_image: Jenkins pipeline execution results showing successful stages)

## Results

This implementation creates a fully automated process for provisioning and configuring AWS EC2 instances. When the Jenkins pipeline runs, it triggers Ansible to ensure the target environments are consistently configured according to specifications.

![Complete Infrastructure](suggested_image: diagram or screenshot showing the fully configured system)

## Future Improvements

- Add more target environments beyond the initial two EC2 instances
- Implement dynamic inventory management
- Enhance error handling and notification systems