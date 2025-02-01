# [Production - Limited Scope]: Patch Management for Differing Linux Distributions
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

## Breakdown of Code

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




