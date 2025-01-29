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
