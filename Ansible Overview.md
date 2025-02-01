# What is Ansible?
Ansible is a highly versatile automation tool that allows for the management of configurations and resources, and deployment of applications.
Ansible allows the usage of the concept of Infrastructure as Code, the idea that an environment and its characteristics/connections can be described as code (YAML) for easy and reliable reproducibility. 
Infrastructure as Code removes a lot of the chances for human error making it essential in any enterprise environment.
## How does it work?
Ansible utilizes an agentless architecture, meaning that no there is no software that needs to be installed on host machines. 
Ansible uses SSH for connections, allowing for the Management Node to execute any command remotely that the ansible remote user has persmissions to.

![image](https://github.com/user-attachments/assets/c1c1392f-a202-4218-8d7b-790e85df76d4)

## Simple Patch Playbook For Differing Linux Distributions
This is the first playbook we created to perform a simple update/upgrade to the hosts declared in our inventory. The primary source for the creation of the playbook was the Ansible official documentation.
```
- name: Update all servers
  hosts: centos, rhel
  become: true
  gather_facts: yes
  remote_user: root

  tasks:
  - name: update RedHat
    ansible.builtin.dnf:
      name: "*"
      state: latest
      update_cache: true
    when: ansible_facts['distribution'] == "RedHat" or ansible_facts['distribution'] == "CentOS"

  - name: update Debian
    ansible.builtin.apt:
      name: "*"
      state: latest
      update_cache: true
    when: ansible_facts['distribution'] == "Debian" or ansible_facts['distribution'] == "Ubuntu"
```
What the playbook looks like when executed correctly. Notice how Ansible will skip the the host when it does not meet the condition stated in ```when:```
![image](https://github.com/user-attachments/assets/029c0c11-1f25-4480-b312-1aa2ab62bc69)

### [hosts:](https://docs.ansible.com/ansible/latest/inventory_guide/intro_patterns.html)
Declares the hosts from the Ansible inventory file as target for the playbook. In this demonstration we use ```all``` , but you can declare a specific host group or a specific IP Address as long as they are located within the default inventory file
```
hosts: ansibleclients
hosts: XXX.XXX.XXX.XXX
```

### [become:](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_privilege_escalation.html)
Allows Ansible to privilege escalate to complete tasks. Can be either ```true``` or ```false```(default).
```
become: true
become: false
```

### [gather_facts:](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html)
Allows Ansible to gather facts (Operating System, IP Address, Hardware, etc.) about the target Managed Host(s). Can be either ```true``` or ```false```. For this playbook, we need operating system information so we can use the appropriate package manager for different Linux distributions.
```
gather_facts: true
gather_facts: false
```

### [remote_user:](https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html)
Declares the user on the Managed Host that Ansible will connect to via SSH.
```
remote_user: root
```

### [ansible.builtin.dnf:](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/dnf_module.html)
Ansible utility that manages ```dnf``` package manager. Has many parameters that can be declared to perform the many functions of ```dnf```. For our environment we are using ```name:``` to declare the package to update, ```state:``` for how to install updated packages, and ```update_cache:``` to have dnf check is package cache is outdated.

### [when:](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_conditionals.html)
Allows the usage of conditionals based on a wide assortment of parameters. In our example, we use ```when:``` to perform tasks based on operation system:
```
when: ansible_facts['distribution']
```
This allows us to either use ```dnf``` or ```apt``` depending on the operating system.


## [WIP] SUF (Splunk Universal Forwarder) Batch Deployment to Linux
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

```
ansible-playbooks sufdeploy.yml
```

![image](https://github.com/user-attachments/assets/5baeec43-7a04-4029-bb17-b6423dfd5fdf)

Below is a picture of the newly added host seen from the perspective of our deployment server:

![image](https://github.com/user-attachments/assets/a2669afe-841d-4d47-a0f0-14b45c19590b)


