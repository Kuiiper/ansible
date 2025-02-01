# What is Ansible?
Ansible is a highly versatile automation tool that allows for the management of configurations and resources, and deployment of applications.
Ansible allows the usage of the concept of Infrastructure as Code, the idea that an environment and its characteristics/connections can be described as code (YAML in this case) for easy and reliable reproducibility. 
Infrastructure as Code removes a lot of the chances for human error making it essential in any enterprise environment.
## How does it work?
Ansible consists of 2 primary components: Control Nodes (server) and Managed Hosts (clients). Ansible uses YAML files known as [playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html) that contain a series of tasks that will be executed against a target [inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)/host. Playbooks are stored in a default location ```/etc/ansible/playbooks``` on the Control Node. There is also an Inventory (Managed Hosts) file stored on the Control Node ```/etc/ansible/hosts```  Ansible utilizes an agentless architecture, meaning that no there is no software that needs to be installed on Managed Host machines. 
Ansible Control Node uses SSH to connect to a user on the Managed Host machine, allowing for the Control Node to execute any command remotely that the remote user has persmissions to.

![image](https://github.com/user-attachments/assets/b5c49ba4-9262-4cc2-9b44-4a12dcd14d79)

## [Baremetal Setup - Linux](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-and-upgrading-ansible-with-pip)

Ansible needs Python3 and the Python3 -pip module. First, check the version of python3 on your machine, update if needed:
```
python3 -V
```
You will need the [```pip```](https://pypi.org/project/pip/) module to install Ansible. Run the following command to download and install pip, or upgrade to latest version if you already have it.
```
python -m ensurepip --upgrade
```
To install Ansible using ```pip``` use the following command:
```
python3 -m pip install --user ansible
```
Once finished Ansible should be ready to use. Ansible will have a default location in ```/etc/ansible```, for our environment, this location was not automatically created. For this we simply created the necessary directories and files for usage.
```
mkdir /etc/ansible
mkdir /etc/ansible/playbooks
touch /etc/ansible/hosts
touch /etc/ansible/ansible.cfg
```




