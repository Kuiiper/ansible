# Overview
This repository outlines the steps followed to create a simple Ansible environment, which includes a Control Node and a Managed Host. This will include some of the troubleshooting processes involved with the setup.
- [Setup](#Setup)
- [Troubleshooting](#Troubleshooting)

# Setup
Red Hat Linux
```
yum install ansible
yum install sshpass
```

# Troubleshooting
Here we will give a general outline of the various problems we encountered while trying to deploy Ansible in our environment. Will include problems in a rough chronological order as encountered beginning with and explanation of the problem followed by the solution/work-around.
## Repository Problem
We encountered an error involving where a previous edit to the default repository for Redhat ``` /etc/yum.repos.d/redhat.repo``` caused us to be unable to install ```sshpass```, which we needed for password usage with Ansible. Simple solution was to re-name the existing ```redhat.repo``` file to ```redhat.bak```:
```
mv redhat.repo redhat.bak
```
Then, we created a new redhat.repo file and updated our repositories
```
touch redhat.repo
sudo yum update
```
This solution ended up working as the new redhat.repo file was populated with the default repository information. Re-naming the original file to a .bak just served as a reminder to us that we would need to revisit the broken repository file to identify the real issue and restore some old configurations.
