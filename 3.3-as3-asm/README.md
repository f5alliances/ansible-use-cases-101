# Exercise 3.3 - Deploying a WAF policy through AS3

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
- [Playbook Output](#playbook-output)
- [Solution](#solution)

# Objective

Demonstrate building an Application Service through an AS3 declaration where a WAF policy gets added.


# Guide

#### Make sure the BIG-IP configuration is clean, run exercise 3.2-as3-delete before proceeding

## Step 1:

In exercise 3.0-as3-intro it is explained how AS3 works.

Throughout this exercise we will use three files.

1. `tenant_base.j2`

<!-- {% raw %} -->
```
{
    "class": "AS3",
    "action": "deploy",
    "persist": true,
    "declaration": {
        "class": "ADC",
        "schemaVersion": "3.2.0",
        "id": "testid",
        "label": "test-label",
        "remark": "test-remark",
        "WorkshopExample":{
            "class": "Tenant",
            {{ as3_app_body }}
        }
    }
}

```
<!-- {% endraw %} -->

 `tenate_base.j2` - is an F5 provided standard template and will include the pointer towards thesecond jinja2 template which includes te actual AS3 schema.

----

2. `as3_template.j2`

<!-- {% raw %} -->
```
"web_app": {
    "class": "Application",
    "template": "http",
    "serviceMain": {
        "class": "Service_HTTP",
        "virtualAddresses": [
            "{{private_ip}}"
        ],
        "pool": "app_pool",
        "policyWAF": {
          "use": "new_asm_policy"
         }
    },
    "app_pool": {
        "class": "Pool",
        "monitors": [
            "http"
        ],
        "members": [
            {
                "servicePort": 80,
                "serverAddresses": [
                    {% set comma = joiner(",") %}
                    {% for mem in pool_members %}
                        {{comma()}} "{{  hostvars[mem]['ansible_host']  }}"
                    {% endfor %}

                ]
            }
        ]
    },
    "new_asm_policy": {
      "class": "WAF_Policy",
      "url": "https://github.com/f5alliances/ansible-use-cases-101/blob/master/3.3-as3-asm/Test_WAF_Policy.xml",
      "ignoreChanges": true
   }
}

```
<!-- {% endraw %} -->


Most of the template is already explained in exercise 3.0-as3-intro. Compared to the original a WAF policy has been added to the as3_template.j2 schema.
The AS3_template.j2 now includes a new class **WAF_Policy** called `new_asm_policy`.
 - `url` defines the URL where to pull the ASM policy from.
 

**COPY THESE TEMPLATES TO YOUR WORKING DIRECTORY**
<!-- {% raw %} -->
```
mkdir waf-j2
cp ~/networking-workshop/3.3-as3-intro/j2/* waf-j2/
```
<!-- {% endraw %} -->

## Step 3:

Using your text editor of choice create a new file called `as3.yml`:

>`vim` and `nano` are available on the control node, as well as Visual Studio and Atom via RDP

## Step 4:

Enter the following play definition into `waf-as3.yml`:
<!-- {% raw %} -->
``` yaml
- name: Deploy WAF profile using AS3
  hosts: lb
  connection: local
  gather_facts: false

  vars:
    pool_members: "{{ groups['webservers'] }}"

```
<!-- {% endraw %} -->


## Step 5

**Append** the following to the waf-as3.yml Playbook.  

<!-- {% raw %} -->
```
  tasks:

  - set_fact:
     provider:
      server: "{{private_ip}}"
      user: "{{ansible_user}}"
      password: "{{ansible_ssh_pass}}"
      server_port: 8443
      validate_certs: no

  - name: Provision BIG-IP with ASM module
    bigip_provision:
      provider: "{{provider}}"
      module: "asm"
      level: "nominal"

```
<!-- {% endraw %} -->

The provider gets set and the ASM module gets provisioned to level 'nominal.

## Step 6
**Append** the following to the waf-as3.yml playbook.

<!-- {% raw %} -->
```
  - name: CREATE AS3 JSON BODY
    set_fact:
      as3_app_body: "{{ lookup('template', 'waf-j2/as3_template.j2', split_lines=False) }}"
```
<!-- {% endraw %} -->

`as3_app_body` will get defined via `set_fact` and renders the as3_template.j2 that is provided.

## Step 6

**Append** the following to the waf-as3.yml Playbook.  

<!-- {% raw %} -->
```
  - name: PUSH AS3
    uri:
      url: "https://{{ ansible_host }}:8443/mgmt/shared/appsvcs/declare"
      method: POST
      body: "{{ lookup('template','waf-j2/tenant_base.j2', split_lines=False) }}"
      status_code: 200
      timeout: 300
      body_format: json
      force_basic_auth: yes
      user: "{{ ansible_user }}"
      password: "{{ ansible_ssh_pass }}"
      validate_certs: no
    delegate_to: localhost

```
<!-- {% endraw %} -->

Pushing AS3 has been explained in exercise 3.0-as3-intro. Basically the `uri` parameter gets used to create the REST body. The declaration uses 'waf-j2/tenant_base.j2' as the body.

## Step 7
Run the playbook - exit back into the command line of the control host and execute the following:

<!-- {% raw %} -->
```
[student1@ansible ~]$ ansible-playbook waf-as3.yml
```
<!-- {% endraw %} -->

# Playbook Output

The output will look as follows.

<!-- {% raw %} -->
```yaml
PLAY [Deploy WAF profile using AS3] ********************************************************************************************************************

TASK [set_fact] ****************************************************************************************************************************************
ok: [f5]

TASK [Provision BIG-IP with ASM module] ****************************************************************************************************************
changed: [f5]

TASK [CREATE AS3 JSON BODY] ****************************************************************************************************************************
ok: [f5]

TASK [PUSH AS3] ****************************************************************************************************************************************
ok: [f5 -> localhost]

PLAY RECAP *********************************************************************************************************************************************
f5                         : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```
<!-- {% endraw %} -->

# Solution

The finished Ansible Playbook is provided here for an Answer key.  Click here: [as3.yml](https://github.com/network-automation/linklight/blob/master/exercises/ansible_f5/3.0-as3-intro/as3.yml).

# Verifying the Solution

Login to the F5 with your web browser to see what was configured.  Grab the IP information for the F5 load balancer from the lab_inventory/hosts file, and type it in like so: https://X.X.X.X:8443/

![f5 gui as3](f5-as3.gif)

1. Click on the Local Traffic on the lefthand menu
2. Click on Virtual Servers.
3. On the top right, click on the drop down menu titled `Partition` and select WorkshopExample
4. The Virtual Server `serviceMain` will be displayed.
5. Click on `serviceMain` and select the tab **Security** and click Policies. The Application Security policy is `enabled` and the used `policy: new_asm_policy`

----

You have finished this exercise.  [Click here to return to the lab guide](../README.md)
