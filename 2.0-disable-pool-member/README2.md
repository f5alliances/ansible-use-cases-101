# Exercise 2.0 - Disabling a pool member

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

(**Note**: This is an alternative exercise compared to the original and does not contain writing Ansible playbooks, instead this exercise uses the original playbook to explain how Ansible can be used to modify pool member status.)
## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
- [Playbook Output](#playbook-output)
- [Solution](#solution)

# Objective

For this last exercise instead of prescriptive step-by-step walkthrough a framework of objectives with hints for each step will be provided.  

Demonstrate the removal of a node from the pool.  Build a Playbook that:
  - Retrieve Facts from BIG-IP for the pools present on the BIG-IP (in our example only one pool is present)
  - Display pools available
  - Store the pool name as a fact
  - Display all the pool members that belong to the pool => IP and port information to the terminal window
  - Prompt the user to disable a particular member or disable all members of the pool
  - Force the appropriate pool members offline
  - Re-enable the pool members

# Guide

## Step 1:

Using your text editor of choice create a new file called `disable-pool-member.yml`.

<!-- {% raw %} -->
```
[student1@ansible ~]$ nano disable-pool-member.yml
```
<!-- {% endraw %} -->

>`vim` and `nano` are available on the control node, as well as Visual Studio and Atom via RDP

## Step 2:

Copy and paste the following play definition into `disable-pool-member.yml`:

<!-- {% raw %} -->
``` yaml
---

- name:  Disabling a pool member
  hosts: f5
  connection: local
  gather_facts: false

```
<!-- {% endraw %} -->

## Step 3

Add a tasks section and set a provider and start gathering pool facts and use the information to collect pool member details and next modify the pool member status.

<!-- {% raw %} -->
```
---
- name: "Disabling a pool member"
  hosts: lb
  gather_facts: false
  connection: local

  tasks:
  - name: Setup provider
    set_fact:
     provider:
      server: "{{private_ip}}"
      user: "{{ansible_user}}"
      password: "{{ansible_ssh_pass}}"
      server_port: "8443"
      validate_certs: "no"

  - name: Query BIG-IP facts
    bigip_device_facts:
      provider: "{{provider}}"
      gather_subset:
       - ltm-pools
    register: bigip_facts

  - name: Display Pools available
    debug: "msg={{item.name}}"
    loop: "{{bigip_facts.ltm_pools}}"
    loop_control:
      label: "{{item.name}}"

  - name: Store pool name in a variable
    set_fact:
     pool_name: "{{bigip_facts.ltm_pools[0].name}}"

  - name: "Show members belonging to pool {{pool_name}}"
    debug: "msg={{item}}"
    loop: "{{bigip_facts.ltm_pools | json_query(query_string)}}"
    vars:
     query_string: "[?name=='{{pool_name}}'].members[*].name[]"

  - pause:
      prompt: "To disable a particular member enter member with format member_name:port \nTo disable all members of the pool enter 'all'"
    register: member_name

  - name: Disable ALL pool members
    bigip_pool_member:
      provider: "{{provider}}"
      state: "forced_offline"
      name: "{{item.split(':')[0]}}"
      pool: "{{pool_name}}"
      port: "{{item.split(':')[1]}}"
      host: "{{hostvars[item.split(':')[0]].ansible_host}}"
    loop: "{{bigip_facts.ltm_pools | json_query(query_string)}}"
    vars:
     query_string: "[?name=='{{pool_name}}'].members[*].name[]"
    when: '"all" in member_name.user_input'

  - name: Disable pool member {{member_name.user_input}}
    bigip_pool_member:
      provider: "{{provider}}"
      state: "forced_offline"
      name: "{{member_name.user_input.split(':')[0]}}"
      pool: "{{pool_name}}"
      port: "{{member_name.user_input.split(':')[1]}}"
      host: "{{hostvars[member_name.user_input.split(':')[0]].ansible_host}}"
    when: '"all" not in member_name.user_input'

```
<!-- {% endraw %} -->

Most of what is happening in the playbook is known by now. Notice the `pause` where the playbook will ask you to make a choice. The options are to either disable all pool members or to select a pool member of choice.


## Step 5
Run the playbook - exit back into the command line of the control host and execute the following:

```
[student1@ansible ~]$ ansible-playbook disable-pool-member.yml
```

# Playbook Output

The output will look as follows.

<!-- {% raw %} -->
```yaml
[student1@ansible ~]$ ansible-playbook disable-pool-member.yml

PLAY [Disabling a pool member] ******************************************************************************************************************************

TASK [Setup provider] *******************************************************************************************************************************
ok: [f5]

TASK [Query BIG-IP facts] ***********************************************************************************************************************************
changed: [f5]

TASK [Display Pools available] ******************************************************************************************************************************
ok: [f5] => (item=http_pool) => {
    "msg": "http_pool"
}

TASK [Store pool name in a variable] ************************************************************************************************************************
ok: [f5] => (item=None)
ok: [f5]

TASK [Show members belonging to pool http_pool] *************************************************************************************************************
ok: [f5] => (item=host1:80) => {
    "msg": "host1:80"
}
ok: [f5] => (item=host2:80) => {
    "msg": "host2:80"
}

TASK [pause] ************************************************************************************************************
[pause]
To disable a particular member enter member with format member_name:port
To disable all members of the pool enter 'all':
host1:80

TASK [Disable ALL pool members] ************************************************************************************************************************
skipping: [f5] => (item=host1:80)
skipping: [f5] => (item=host2:80)

TASK [Disable pool member host1:80] *************************************************************************************************************************
changed: [f5]

PLAY RECAP **************************************************************************************************************
f5                         : ok=7    changed=2    unreachable=0    failed=0
```
<!-- {% endraw %} -->

# Solution

The finished Ansible Playbook is provided here for an Answer key.  Click here: [disable-pool-member.yml](../2.0-disable-pool-member/disable-pool-member.yml).

# Verifying the Solution

The playbook output shows that disabling of host1:80 was selected. Your choice can be different, To see the disabled **pool member(s)**, login to the F5 load balancer with your web browser.

>Grab the IP information for the F5 load balancer from the `/home/studentX/networking_workshop/lab_inventory/hosts` file, and type it in like so: https://X.X.X.X:8443/

Login information for the BIG-IP:
- username: admin
- password: **provided by instructor** defaults to ansible

The load balancer virtual server can be found by navigating the menu on the left.  Click on **Local Traffic**. then click on **Pools**, select **http_pool** and check out the **Members**.

# Going Further

Let's change the pool state back to its origin state.

## Step 6

```
[student1@ansible ~]$ nano disable-pool-member.yml
```

Go to the section after `pause` where the choice of diabling pool members is made. Within the Ansible network module called `bigip_pool_member`, change the parameter `state` to enabled and save your changes.

## Step 7

Re-run the playbook:

```
[student1@ansible ~]$ ansible-playbook disable-pool-member.yml
```

Be sure to `enable` what you have disabled in the previous action.

# Verifying the Solution

To see the enabled **pool member(s)**, login to the F5 load balancer with your web browser.

>Grab the IP information for the F5 load balancer from the `/home/studentX/networking_workshop/lab_inventory/hosts` file, and type it in like so: https://X.X.X.X:8443/

Login information for the BIG-IP:
- username: admin
- password: **provided by instructor** defaults to ansible

The load balancer virtual server can be found by navigating the menu on the left.  Click on **Local Traffic**. then click on **Pools**, select **http_pool** and check out the **Members**.


--
You have finished this exercise.  [Click here to return to the lab guide](../README.md)
