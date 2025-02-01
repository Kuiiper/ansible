# What is Ansible?
Ansible is a highly versatile automation tool that allows for the management of configurations and resources, and deployment of applications.
Ansible allows the usage of the concept of Infrastructure as Code, the idea that an environment and its characteristics/connections can be described as code (YAML) for easy and reliable reproducibility. 
Infrastructure as Code removes a lot of the chances for human error making it essential in any enterprise environment.
## How does it work?
Ansible utilizes an agentless architecture, meaning that no there is no software that needs to be installed on host machines. 
Ansible uses SSH for connections, allowing for the Management Node to execute any command remotely that the ansible remote user has persmissions to.

![image](https://github.com/user-attachments/assets/c1c1392f-a202-4218-8d7b-790e85df76d4)
## Execution
```
ansible-playbook playbook.yml
```
## Simple Patch Playbook
This is the first playbook I created to perform a simple update/upgrade to the hosts declared in our inventory. The primary source for the creation of the playbook was the Ansible official documentation.
```
name: Update all servers
hosts: all
become: true
gather_facts: yes
remote_user: ansible

  tasks:
  
name: update RedHat
  ansible.builtin.dnf:
    name: "*"
    state: latest
    update_cache: true
  when: ansible_facts['distribution'] == "RedHat" or ansible_facts['distribution'] == "CentOS"

  
name: update Debian
  ansible.builtin.apt:
    name: "*"
    state: latest
    update_cache: true
  when: ansible_facts['distribution'] == "Debian"
```
## [WIP] Demo SUF (Splunk Universal Forwarder) Batch Deployment to Linux
In our test environment, we were able deploy a SUF instance to a Managed Host and have it connect back to the deployment server with no manual interaction. With this project, we will be able to batch deploy SUFs to the ~100 Linux VMs in our environment.
```
- name: Batch Deploy SUF
  hosts: test
  gather_facts: yes
  remote_user: ansible

  tasks:
  - name: Download SUF
    get_url:
      url: https://download.splunk.com/products/universalforwarder/releases/9.4.0/linux/splunkforwarder-9.4.0-6b4ebe426>
      dest: /opt
      mode: '0755'
    register: download

  - name: Install SUF
    ansible.builtin.apt:
      deb: /opt/splunkforwarder-9.4.0-6b4ebe426ca6-linux-amd64.deb
      state: present

  - name: Starting Splunk
    ansible.builtin.command: /opt/splunkforwarder/bin/splunk start --accept-license --answer-yes --no-prompt --seed-passwd ************

  - name: Connect Client
    command: /opt/splunkforwarder/bin/splunk set deploy-poll XXX.XXX.XXX.XXX:8089 -auth admin:************

  - name: restart client
    command: /opt/splunkforwarder/bin/splunk restart
```
What running the playbook looks like in the terminal:

![image](https://github.com/user-attachments/assets/5baeec43-7a04-4029-bb17-b6423dfd5fdf)

Below will be a picture of the newly added host seen from the perspective of our deployment server:

![image](https://github.com/user-attachments/assets/a2669afe-841d-4d47-a0f0-14b45c19590b)


