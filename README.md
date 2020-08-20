# Ansible_AWS
# Guide covers creating an EC2 instance on AWS using Ansible

- To start this guide, go ahead and create a Virtual Machine to host our Ansible (Call this machine ansible helps)

#### Ansible machine Pre-requisites

1. sudo apt update
2. sudo apt install software-properties-common
3. sudo apt-add-repository --yes --update ppa:ansible/ansible
4. sudo apt install ansible 

- You can check ansible is installed with `ansible --version`

## Guide part 1 - Install the EC2 module dependencies

1. sudo apt install python

2. sudo apt install python-pip -y

3. sudo pip install --upgrade pip

4. sudo pip install boto

5. sudo pip install boto3

## Guide part 2 - Create SSH keys to connect to the EC2 instance

- ssh-keygen -t rsa -b 4096 -f ~/.ssh/`<Your_Name>`_aws

## Guide part 3 - Create Ansible Directory Structure

```shell script
mkdir -p AWS_Ansible/group_vars/all/
cd AWS_Ansible
touch playbook.yml
```

## Guide part 4 - Create Ansible Vault file to store the AWS Access and Secret keys

- `ansible-vault create group_vars/all/pass.yml`
- Make your vault password

## Guide part 5 - Edit the vault and store your ec2 secret and access key

- `ansible-vault edit group_vars/all/pass.yml`
- Append to the file. Insert:
```yaml
ec2_access_key: <Your_Access_Key>                                     
ec2_secret_key: <Your_Secret_Key>
```

## Guide part 6 - Open the playbook.yml and add the following

```yaml

# AWS playbook
---
- hosts: aws
  gather_facts: False

  vars:
    key_name: Ib_Bo_aws
    region: eu-west-1
    image: ami-05ec38f16b5a19c94
    id: "Eng67.Ib.App.Ansible"
    sec_group: "<Enter your premade security group id here>"
    subnet_id: "<enter the subnet_id here>"

  tasks:

    - name: Facts
      block:

      - name: Get instances facts
        ec2_instance_facts:
          aws_access_key: "{{ec2_access_key}}"
          aws_secret_key: "{{ec2_secret_key}}"
          region: "{{ region }}"
        register: result

      tags: always


    - name: Provisioning EC2 instances
      block:

      - name: Upload public key to AWS
        ec2_key:
          name: "{{ key_name }}"
          key_material: "{{ lookup('file', '~/.ssh/{{ key_name }}.pub') }}"
          region: "{{ region }}"
          aws_access_key: "{{ec2_access_key}}"
          aws_secret_key: "{{ec2_secret_key}}"

      - name: Provision instance(s)
        ec2:
          aws_access_key: "{{ec2_access_key}}"
          aws_secret_key: "{{ec2_secret_key}}"
          assign_public_ip: true
          key_name: "{{ key_name }}"
          id: "{{ id }}"
          group_id: "{{ sec_group }}"
          vpc_subnet_id: "{{ subnet_id }}"
          image: "{{ image }}"
          instance_type: t2.micro
          region: "{{ region }}"
          wait: true
          count: 1
          # exact_count: 2
          count_tag:
            instance_tags:
              Name: "{{ id }}"

      tags: ['never', 'create_ec2']



      - name: Start instance
      - ec2_instance:
        state: restarted
        instance_ids:
          - <enter instance id here>

      tags: ['never', 'start_ec2']

```

- Edit the playbook to fulfil your needs

## Guide part 7 - Create the EC2 Instance

```shell script
ansible-playbook playbook.yml --ask-vault-pass --tags create_ec2
```
## Guide part 8
