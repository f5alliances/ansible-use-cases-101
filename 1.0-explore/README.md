### Exercise 1.0: Exploring the lab environment

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

Welcome to this Ansible 101 training.

This training will take you through the basic knowledge and exercises to gain understanding how F5 BIG-IP can leverage Ansible to facilitate automated application services.

Being a '101', this training will only cover:

 - Exploring some Ansible basics.
 - Gathering BIG-IP facts. 
 - Creating BIG-IP objects like nodes, pool, virtual server and iRules.
 - Disable pool member, delete configuration and error handling.
 - Introduction of using AS3 with Ansible.

After taking this course a student should be able to create simple playbooks for deploying application services on BIG-IP and be able to use AS3 with Ansible and push configurations through a playbook on the BIG-IP.

 `Note: If you have any questions or issues, don't hesitate to open an issue to ask for help!` [Click here to open an issue](https://github.com/f5alliances/ansible-use-cases-101/issues).

#### Pre-requisites
To get the most out of this training, the student should already have some basic knowledge about the use of Ansible and BIG-IP.
When you are completely new to Ansible it is recommended to start with the 2 hour [Ansible Quick Start](https://linuxacademy.com/cp/modules/view/id/288) training from Linux Academy.

#### Introduction
Ansible is an open-source automation engine for software provisioning, configuration management and application-deployment. It allows for agent-less system configuration, which means it does not deploy agents to nodes. Communication to those nodes works by using SSH and Python is a requirement on those managed nodes. This makes that communication to managed nodes can be secure. Python is used to support the modules which support the tasks which will get deployed by Ansible through the use of so-called playbooks. 

When Ansible gets installed it will contain an initial 'Ansible configuration file' called ansible.cfg file which includes the inventory of hosts and where the library of modules can be found.

Let's start to explore by walking through the steps.

#### Step 1

Navigate to the `networking-workshop` directory.

```
[student1@ansible ~]$ cd networking-workshop/
[student1@ansible networking-workshop]$
```

#### Step 2

Run the `ansible` command with the `--version` command to look at what is configured:

```
[student1@ansible networking-workshop]$ ansible --version
ansible 2.6.2
  config file = /home/student1/.ansible.cfg
  configured module search path = [u'/home/student1/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, May  3 2017, 07:55:04) [GCC 4.8.5 20150623 (Red Hat 4.8.5-14)]
```

> Note: The Ansible version you see might differ from the above output

This command gives you information about the version of Ansible, location of the executable, version of Python, search path for the modules and location of the `ansible configuration file`.

#### Step 3

Use the `cat` command to view the contents of the `ansible.cfg` file.


```
[student1@ansible networking-workshop]$ cat ~/.ansible.cfg
[defaults]
connection = smart
timeout = 60
inventory = /home/student1/networking-workshop/lab_inventory/hosts
host_key_checking = False
private_key_file = /home/student1/.ssh/aws-private.pem
[student1@ansible networking-workshop]$

```

Note the following parameters within the `ansible.cfg` file:

 - `inventory`: shows the location of the Ansible inventory being used.
 - `private_key_file`: this shows the location of the private key used to login to devices

#### Step 4

The scope of a `play` within a `playbook` is limited to the groups of hosts declared within an Ansible **inventory**. Ansible supports multiple [inventory](http://docs.ansible.com/ansible/latest/intro_inventory.html) types. An inventory could be a simple flat file with a collection of hosts defined within it or it could be a dynamic script (potentially querying a CMDB backend) that generates a list of devices to run the playbook against.

In this lab you will work with a file based inventory written in the **ini** format. Use the `cat` command to view the contents of your inventory:

`[student1@ansible networking-workshop]$ cat lab_inventory/hosts`

The output will look as follows with student2 being the respective student workbench:
```
[all:vars]
ansible_user=student2
ansible_ssh_pass=ansible
ansible_port=22

[lb]
f5 ansible_host=34.199.128.69 ansible_user=admin private_ip=172.16.26.136 ansible_ssh_pass=admin

[control]
ansible ansible_host=107.23.192.217 ansible_user=ec2-user private_ip=172.16.207.49

[webservers]
host1 ansible_host=107.22.141.4 ansible_user=ec2-user private_ip=172.16.170.190
host2 ansible_host=54.146.162.192 ansible_user=ec2-user private_ip=172.16.160.13
```

#### Step 5

In the above output every `[ ]` defines a group. For example `[webservers]` is a group that contains the hosts `host1` and `host2`.

> Note: A group called **all** always exists and contains all groups and hosts defined within an inventory.


We can associate variables to groups and hosts. Host variables are declared/defined on the same line as the host themselves. For example for the host `f5`:

```
f5 ansible_host=34.199.128.69 ansible_user=admin private_ip=172.16.26.136 ansible_ssh_pass=admin
```

 - `f5` - The name that Ansible will use.  This can but does not have to rely on DNS
 - `ansible_host` - The IP address that ansible will use, if not configured it will default to DNS
 - `ansible_user` - The user ansible will use to login to this host, if not configured it will default to the user the playbook is run from
 - `private_ip` - This value is not reserved by ansible so it will default to a [host variable](http://docs.ansible.com/ansible/latest/intro_inventory.html#host-variables).  This variable can be used by playbooks or ignored completely.
- `ansible_ssh_pass` - The password ansible will use to login to this host, if not configured it will assume the user the playbook ran from has access to this host through SSH keys.  

> Does the password have to be in plain text?  No, Red Hat Ansible Tower can take care of credential management in an easy to use web GUI or a user may use [ansible-vault](https://docs.ansible.com/ansible/latest/network/getting_started/first_inventory.html#protecting-sensitive-variables-with-ansible-vault)

Go back to the home directory

```
[student1@ansible networking-workshop]$ cd ~
```

You have finished this exercise.  [Click here to return to the lab guide](../README.md)

