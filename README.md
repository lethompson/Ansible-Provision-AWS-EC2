# Provision EC2 instances with Ansible

### Prequisites for integration of Ansible and AWS Cloud:
* Create AWS user
* Install Ansible and Ansible EC2 module dependencies
* Create SSH keys via ssh-keygen


### Install Ansible and the EC2 module dependencies
```
sudo apt install python
sudo apt install python-pip
pip install python-boto python-boto3 ansible

```
#### Note: Demo based on Ansible version 2.5.1 and Python version 2.7

### Create SSH keys to connect toe the EC2 instance after provision via Ansible

```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_aws
```

### Create the Ansible directory structure

```
.
├── group_vars
│   └── all
│       └── pass.yml
├── playbook.yml

```

```
mkdir -p AWS_Ansible_Config/groups_vars/all/
cd AWS_Ansible_Config
touch playbook.yml
```
