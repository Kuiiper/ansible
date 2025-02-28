# [WIP - Testing] initUser Framework for Ansible Deployment
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
      key: "{{ lookup('file', '/root/.ssh/ansibleKey.pub') }}"

  - name: Delete initUser
    ansible.builtin.user
      user: ansible
      state: absent
```
