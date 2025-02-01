# [WIP - Not Production]: SUF ([Splunk Universal Forwarder](https://www.splunk.com/en_us/blog/learn/splunk-universal-forwarder.html)) Batch Deployment to Linux
In our test environment, we were able deploy a SUF instance to a Managed Host and have it connect back to the deployment server with no manual interaction. With this project, we will be able to batch deploy SUFs to the ~100 Linux VMs in our environment. The end goal is to create a highly secured, shared Ansible user that we can utilize to deploy applications to in batches.
```
- name: Batch Deploy SUF
  hosts: test
  gather_facts: yes
  remote_user: ansible

  tasks:
  - name: Download SUF
    get_url:
      url: https://download.splunk.com/products/universalforwarder/releases/9.4.0/linux/splunkforwarder-9.4.0-6b4ebe426ca6-linux-amd64.deb
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

## Breakdown of Code

### [hosts:](https://docs.ansible.com/ansible/latest/inventory_guide/intro_patterns.html)
Declares the hosts from the Ansible inventory file as target for the playbook. In this demonstration we use ```all``` , but you can declare a specific host group or a specific IP Address as long as they are located within the default inventory file.
```
hosts: examplegroup
hosts: X.X.X.X
```

### [gather_facts:](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html)
Allows Ansible to gather facts (Operating System, IP Address, Hardware, etc.) about the target Managed Host(s). Can be either ```true``` or ```false```. We plan on implementing accounting for differing OS as part of a later update. Will need information for ```when:``` conditional.
```
gather_facts: true
gather_facts: false
```

### [remote_user:](https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html)
Declares the user on the Managed Host that Ansible will connect to via SSH.
```
remote_user: example_user
```

### [get_url:](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/get_url_module.html)
Allows you to download a file from a specified URL. Works exactly like running ```wget``` manually. ```dest:``` specifies the destination for the downloaded file. ```mode:``` allows you to set the permissions of the downloaded file for users and groups. ```register:```Conditional that registers the output of the task as a variable that can be invoked later.
```
get_url:
      url: https://download.example.com/exampleapplication.deb
      dest: /example_user/dhome
```
### [ansible.builtin.apt:](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)
Ansible utility that manages ```apt``` package manager. Has many parameters that can be declared to perform the many functions of ```apt```. In Ansible, the path to a .deb file must be denoted using ```deb:``` instead of ```name:```.

### [command:](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html)
Allows you to execute commands any commands you can normally perform in a shell. The commands we use for the last 3 steps are all steps that are normally done manually during the installation process of the SUF. Ansible allows us to deploy these commands to all targeted Managed Hosts.
