# [WIP] SUF ([Splunk Universal Forwarder](https://www.splunk.com/en_us/blog/learn/splunk-universal-forwarder.html)) Batch Deployment to Linux
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

