# Provision EC2 instances with Ansible

### Prequisites for integration of Ansible and AWS Cloud:
* Create AWS user
* Install Ansible and Ansible EC2 module dependencies
* Create SSH keys via ssh-keygen


### Install Ansible and the EC2 module dependencies
```
> sudo apt install python
> sudo apt install python-pip
> pip install python-boto python-boto3 ansible

```
#### Note: Demo based on Ansible version 2.5.1 and Python version 2.7

### Create SSH keys to connect toe the EC2 instance after provision via Ansible

```
> ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_aws
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
> mkdir -p AWS_Ansible_Config/groups_vars/all/
> cd AWS_Ansible_Config
> touch playbook.yml
```

### Create Ansible Vault file to store the AWS Access and Secret keys to have IAM permission for User to provision EC2 instance with Ansible

```
> ansible-vault create group_vars/all/pass.yml
New Vault password:
Confirm New Vault password:
```

#### Note: The password provided will be asked every time the playbook is executed or when editing the pass.yml file via ansible-vault

### Edit the pass.yml file and create the keys global constants
* Create the variables ec2_access_key, ec2_secret_key
* Set the values gathered after user creation (IAM)

```
> ansible-vault edit group_vars/all/pass.yml
Vault password:

ec2_access_key: <ACCESS KEY>
ec2_secret_key: <SECRET KEY>
```

### Open the playbook.yml file and past the following content:
```
# AWS playbook
---

- hosts: localhost
  connection: local
  gather_facts: False


  vars:
    key_name: my_aws
    region: us-east-1
    image: ami-b63769a1 
    id: "web-app"
    sec_group: "{{ id }}-sec"

  tasks:

    - name: Facts
      block:

      - name: Get instances facts
        ec2_instance_facts:
          aws_access_key: "{{ec2_access_key}}"
          aws_secret_key: "{{ec2_secret_key}}"
          region: "{{ region }}"
        register: result

      - name: Instances ID
        debug:
          msg: "ID: {{ item.instance_id }} - State: {{ item.state.name }} - Public DNS: {{ item.public_dns_name }}"
        loop: "{{ result.instances }}"

      tags: always

    - name: Provisioning EC2 instances
      block:


      - name: Upload public key to AWS
        ec2_key:
          name: "{{ key_name }}"
          key_material: "{{ lookup('file','/home/thompsonl/.ssh/{{ key_name }}.pub') }}"
          region: "{{ region }}"
          aws_access_key: "{{ec2_access_key}}"
          aws_secret_key: "{{ec2_secret_key}}"

      - name: Create security group
        ec2_group:
          name: "{{ sec_group }}"
          description: "Security group for app {{ id }}"
          region: "{{ region }}"
          aws_access_key: "{{ec2_access_key}}"
          aws_secret_key: "{{ec2_secret_key}}"
          rules:
            - proto: tcp
              ports:
                - 22
              cidr_ip: 0.0.0.0/0
              rule_desc: allow all on ssh port
        register: result_sec_group

      - name: Provision instance(s)
        ec2:
          aws_access_key: "{{ec2_access_key}}"
          aws_secret_key: "{{ec2_secret_key}}"
          key_name: "{{ key_name }}"
          id: "{{ id }}"
          group_id: "{{ result_sec_group.group_id }}"
          image: "{{ image }}"
          instance_type: t2.micro
          region: "{{ region }}"
          wait: true
          count: 1


      tags: ['never', 'create_ec2']
```
