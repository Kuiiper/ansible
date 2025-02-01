# What is Ansible?
Ansible is a highly versatile automation tool that allows for the management of configurations and resources, and deployment of applications.
Ansible allows the usage of the concept of Infrastructure as Code, the idea that an environment and its characteristics/connections can be described as code (YAML in this case) for easy and reliable reproducibility. 
Infrastructure as Code removes a lot of the chances for human error making it essential in any enterprise environment.
## How does it work?
Ansible consists of 2 primary components: Control Nodes (server) and Managed Hosts (clients). Ansible uses YAML files known as [playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html) that contain a series of tasks that will be executed against a target [inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)/host. Playbooks are stored in a default location ```/etc/ansible/playbooks``` on the Control Node. There is also an Inventory (Managed Hosts) file stored on the Control Node ```/etc/ansible/hosts```  Ansible utilizes an agentless architecture, meaning that no there is no software that needs to be installed on Managed Host machines. 
Ansible Control Node uses SSH to connect to a user on the Managed Host machine, allowing for the Control Node to execute any command remotely that the remote user has persmissions to.

![image](https://github.com/user-attachments/assets/b5c49ba4-9262-4cc2-9b44-4a12dcd14d79)



# Current Projects [WIP & Complete]
This section outlines our current and in-progress playbooks that we have implemented or plan to implement in our production environment. These projects will range from basic host management, to more advance application deployment and security automations.

## Patch Playbook For Differing Linux Distributions
This playbook is for patch management, the ```tasks:``` in this playbook are the equivalent of running a simple 'update -> upgrade' locally. We use the ```when:``` conditional to make sure Ansible runs the correct package management function against the correct OS. The primary source for the creation of the playbook was the [Ansible community documentation](https://docs.ansible.com/).
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
What the playbook looks like when executed correctly. Notice how each task will skip the host if it does not meet the condition stated in the ```when:``` conditional.

![image](https://github.com/user-attachments/assets/029c0c11-1f25-4480-b312-1aa2ab62bc69)

### [hosts:](https://docs.ansible.com/ansible/latest/inventory_guide/intro_patterns.html)
Declares the hosts from the Ansible inventory file as target for the playbook. In this demonstration we use ```all``` , but you can declare a specific host group or a specific IP Address as long as they are located within the default inventory file.
```
hosts: examplegroup
hosts: X.X.X.X
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
Ansible utility that manages ```dnf``` package manager. Has many parameters that can be declared to perform the many functions of ```dnf```. For our environment we are using ```name:``` to declare the package to update, ```state:``` for how to install updated packages, and ```update_cache:``` to have dnf check if package cache is outdated.

### [when:](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_conditionals.html)
Allows the usage of conditionals based on a wide assortment of parameters. In our example, we use ```when:``` to perform tasks based on operation system:
```
when: ansible_facts['distribution']
```
This allows us to either use ```dnf``` or ```apt``` depending on the operating system.


## [WIP] SUF ([Splunk Universal Forwarder](https://www.splunk.com/en_us/blog/learn/splunk-universal-forwarder.html)) Batch Deployment to Linux
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

Below is a picture of the successfully added host seen from the perspective of our Splunk Deployment Server. Our new host 10.128.69.239 has been added to our list of forwarders:

![image](https://github.com/user-attachments/assets/a2669afe-841d-4d47-a0f0-14b45c19590b)
As you can see, the commands used to 'download -> install -> configure' the application are very similar to how it would be done manually. This process framework can be applied to an assortment of different applications, you would just need to change out the Splunk specific information for information relevant to your needed application.


