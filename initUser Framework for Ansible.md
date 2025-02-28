# [WIP - Testing]: initUser Framework for Ansible Deployment
initUser Framework is the name i am giving to an application of what I would consider some good practices for deploying Ansible throughout an environment. It begins with the addition of a privileged user called initUser, standing for initialization user. The idea of the user is that it will be used as a initial access point for Ansible to begin configuring the chosen host.
Once that purpose has been fulfilled, the user will be deleted.

## Deployment Pre-requisites
These pre-requisites should be added to the golden image/template of every machine you intend on adding Ansible to.
1. initUser with chosen default credentials and sudo/admin access
2. openssh-server for initial connetion
```
sudo apt install openssh-server
sudo systemctl start ssh
```
3. sshpass for automated ssh connections using passwords
```
sudo apt install sshpass
```
## Playbook
```
- name: Ansible Initialization
  hosts: your_host
  become: true
  gather_facts: yes

  tasks:
  - name: Make ansible user
    ansible.builtin.user:
      name: ansible
      shell: /bin/bash
      create_home: yes
      group: sudo

  - name: Deploy SSH Key
    authorized_key:
      user: ansible
      state: present
      key: "{{ lookup('file', '/root/.ssh/id_rsa.pub') }}"

  - name: Delete initUser
    ansible.builtin.user
      user: initUser
      state: absent
```
## Breakdown of Code

### [ansible.builtin.user:](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html)
Module that allows for the manipulation of users on the host. In this case we are using ```name:``` to decalre the name of the new user to be created. the ```shell:``` arguement is used to pass the desired shell of the new user. You can add many other parameters to the new user. The last command of the playbook uses the module as well, but to delete the initUser account used to kickstart the Ansible connection.
```
ansible.builtin.user
  name: new_user
  shell: bin/bash
```
### [authorized_keys:](https://docs.ansible.com/ansible/latest/collections/ansible/posix/authorized_key_module.html)
A module not part of the ```ansible-core``` package. Can install using:
```
ansible-galaxy collection install ansible.posix
```
This module allows you to manage the key exchange for SSH connections. In this instance, we have already generated an SSH keypair using ```ssh key-gen``` and have saved the key as id_rsa for example purposes. We have designated the new user ansible as the target out the exchange. This command we are running is the same as running ```ssh-copy-id -i /path/to/your/key user@X.X.X.X```. ```key:``` is used to define where the key may be stored and the path/url to that key.
