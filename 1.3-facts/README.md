# Exercise 1.3: Ansible Facts

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png) [日本語](README.ja.md).

## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
- [Takeaways](#takeaways)
- [Solution](#solution)

# Objective

Demonstration use of Ansible facts on network infrastructure.

Ansible facts are information derived from speaking to the remote network elements.  Ansible facts are returned in structured data (JSON) that makes it easy manipulate or modify.  For example a network engineer could create an audit report very quickly using Ansible facts and templating them into a markdown or HTML file.

This exercise will cover:
- Building an Ansible Playbook from scratch.
- Using [ansible-doc](https://docs.ansible.com/ansible/latest/cli/ansible-doc.html).
- Using the [ios_facts module](https://docs.ansible.com/ansible/latest/modules/ios_facts_module.html).
- Using the [debug module](https://docs.ansible.com/ansible/latest/modules/debug_module.html).

# Guide

#### Step 1

Using your favorite text editor create a new file called `facts.yml` in the network-workshop directory:  

Enter the following play definition into `facts.yml`:

```yaml
---
- name: gather information from routers
  hosts: cisco
  gather_facts: no
```

Here is an explanation of each line:
- The first line, `---` indicates that this is a YAML file.
- The `- name:` keyword is an optional description for this particular Ansible Playbook.
- The `hosts:` keyword means this playbook against the group `cisco` defined in the inventory file.
- The `gather_facts: no` is required for Ansible 2.8 and earlier


#### Step 2

Next, add the first `task`. This task will use the `ios_facts` module to gather facts about each device in the group `cisco`.


```yaml
---
- name: gather information from routers
  hosts: cisco
  gather_facts: no

  tasks:
    - name: gather router facts
      ios_facts:
```

>A play is a list of tasks. Modules are pre-written code that perform the task.

#### Step 3

Execute the Ansible Playbook:

```
[student1@ansible network-workshop]$ ansible-playbook facts.yml
```

The output should look as follows.

```bash
[student1@ansible network-workshop]$ ansible-playbook facts.yml

PLAY [gather information from routers] *****************************************

TASK [gather router facts] *****************************************************
ok: [rtr1]

PLAY RECAP ******************************************************************************************************************
rtr1                       : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


#### Step 4

The play ran successfully and executed against the Cisco router(s). But where is the output? Re-run the playbook using the verbose `-v` flag.


```
[student1@ansible network-workshop]$ ansible-playbook facts.yml -v
Using /home/student1/.ansible.cfg as config file

PLAY [gather information from routers] *****************************************

TASK [gather router facts] *****************************************************
ok: [rtr1] => changed=false
  ansible_facts:
    ansible_net_all_ipv4_addresses:
    - 192.168.35.101
    - 172.16.129.86
    - 192.168.1.101
    - 10.1.1.101
    - 10.200.200.1
    - 10.100.100.1
.
.
 <output truncated for readability>
.
.
    ansible_net_iostype: IOS-XE
    ansible_net_memfree_mb: 1853993
    ansible_net_memtotal_mb: 2180495
    ansible_net_neighbors: {}
    ansible_net_python_version: 2.7.5
    ansible_net_serialnum: 91Y8URJWFPU
    ansible_net_system: ios
    ansible_net_version: 16.09.02
    discovered_interpreter_python: /usr/bin/python

PLAY RECAP *********************************************************************
rtr1                       : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


> Note: The output returns key-value pairs that can then be used within the playbook for subsequent tasks. Also note that all variables that start with **ansible_** are automatically available for subsequent tasks within the play.

#### Step 5

Running a playbook in verbose mode is a good option to validate the output from a task. To work with the variables within a playbook you can use the `debug` module.

Write two additional tasks that display the routers' OS version and serial number.

<!-- {% raw %} -->
``` yaml
---
- name: gather information from routers
  hosts: cisco
  gather_facts: no

  tasks:
    - name: gather router facts
      ios_facts:

    - name: display version
      debug:
        msg: "The IOS version is: {{ ansible_net_version }}"

    - name: display serial number
      debug:
        msg: "The serial number is:{{ ansible_net_serialnum }}"
```
<!-- {% endraw %} -->


#### Step 6

Now re-run the playbook but this time do not use the `verbose` flag:

```
[student1@ansible network-workshop]$ ansible-playbook facts.yml

PLAY [gather information from routers] **************************************************************************************

TASK [gather router facts] **************************************************************************************************
ok: [rtr1]

TASK [display version] ******************************************************************************************************
ok: [rtr1] =>
  msg: 'The IOS version is: 16.09.02'

TASK [display serial number] ************************************************************************************************
ok: [rtr1] =>
  msg: The serial number is:91Y8URJWFPU

PLAY RECAP ******************************************************************************************************************
rtr1                       : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


Using less than 20 lines of "code" you have just automated version and serial number collection. Imagine if you were running this against your production network! You have actionable data in hand that does not go out of date.

# Takeaways

- The [ansible-doc](https://docs.ansible.com/ansible/latest/cli/ansible-doc.html) command will allow you access to documentation without an internet connection.  This documentation also matches the version of Ansible on the control node.
- The [ios_facts module](https://docs.ansible.com/ansible/latest/modules/ios_config_module.html) gathers structured data specific for Cisco IOS.  There are relevant modules for each network platform.  For example there is a junos_facts for Juniper Junos, and a eos_facts for Arista EOS.
- The [debug module](https://docs.ansible.com/ansible/latest/modules/debug_module.html) allows an Ansible Playbook to print values to the terminal window.


# Solution

The finished Ansible Playbook is provided here for an answer key: [facts.yml](facts.yml).

---

# Complete


----
**Navigation**
<br>
[Previous Exercise](../1.2-first-playbook) - [Next Exercise](../1.4-jinja)

[Click here to return to the Ansible Network Automation Workshop](../README.md)
