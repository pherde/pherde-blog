---
title: "Ansible: A Beginner's Tutorial - Part 1."
date: 2023-03-19
description: Hands on with Ansible basic concepts. Let's create an inventory and a playbook and play a little bit by automating tasks in virtual machines.
draft: false
tags: [ansible, devops]
---

## A little explanation:

First of all, this article/tutorial was created with the intention of reinforcing some of what I learned in KodeKloud's Ansible basics course. That said, let's go on.


## Basic concepts

First let's see what a famous AI says about Ansible:

Ansible is an open-source automation tool used for IT tasks such as configuration management, application deployment, and task automation. It allows users to automate repetitive tasks across multiple servers and devices simultaneously, reducing the need for manual intervention and streamlining workflows.

Okay, with the definition of what Ansible is, let's move on to two basic concepts: **Inventory** and **Playbook**.

**Inventory:** An inventory is a file that defines a list of remote hosts or network devices that Ansible can connect to and manage. The inventory file is a simple text file that contains a list of hosts or IP addresses, grouped by different categories or groups. Usually the inventory is in the format of an .ini file.

**Playbook:** A playbook is a file containing a series of tasks and instructions written in YAML format that are executed on one or more remote hosts. A playbook describes the desired state of a system, which is achieved by executing a series of tasks that perform specific actions, such as installing software, configuring services, or modifying system settings. Playbooks are a key component of Ansible, providing a simple and flexible way to automate complex tasks and manage system configurations. They are designed to be easy to read, write, and maintain.

Okay, with the basic concepts described, let's practice a little.

## Hands on


For our tests we initially need two virtual machines with an installed system, I chose to use **QEMU** as my VM and install the **Ubuntu Server 22.04** system in both VMs. But everything we do here can be done with Virtbualbox and other OS, like Fedora, for example. The important thing is to install the system and be able to access it via ssh. So, our scenario will be:

| Host (with Ansible) | VMs |
| --------------------|-----|
| Ubuntu 20.04        | Ubuntu Server 1 |
|                     | Ubuntu Server 2 |

I recommend using **virt-manager** on Ubuntu, it is the simplest way to work with QEMU in my opinion.

Don't forget to install Ansible in the host machine, in my case in ubuntu just:

```shell
$sudo apt install ansible
```

### Inventory

Remember, the inventory contains the list of hosts that Ansible will access to make its magic.

With our VMs running we are going to write our inventory, so for this we will create a file called `inventory` with the following content:

```ini
vm1 ansible_host=192.168.122.90 ansible_connection=ssh ansible_user=fernando ansible_ssh_pass=mypass!12
vm2 ansible_host=192.168.122.91 ansible_connection=ssh ansible_user=fernando ansible_ssh_pass=mypass!12

[servers]
vm1
vm2
```

**Explaining:**

vm1, vm2 = name, like a variable, for our target host.

ansible_host = address of our target.

ansible_connection = connection type.

ansible_user = name of the user who will access the machine.

ansible_ssh_pass = user password.

[servers] = group name, we can add as many hosts as we want into the group.

I know... putting a password in a text file is not at all safe. But remember, I am only following what I saw in the KodeKloud course. In a second moment, we can use ssh-key instead of a password.

Well, putting the ssh password in the inventory brings a first problem, if you try to ping the VMs via Ansible and you didn't previously access the VMs via standard ssh, Ansible will return an error message like this:

```shell
"Using a SSH password instead of a key is not possible because Host Key
checking is enabled and sshpass does not support this.  Please add this 
host's fingerprint to your known_hosts file to manage this host."
```

We have two solutions for this, the first is to access the VM by ssh in the standard way (*ssh user@ip*). The second solution is to create a configuration file for Ansible to ignore this. For this second solution we will create a file called `ansible.cfg` in the same directory we are working in (the directory where our inventory is) and put the following content:

```ini
[defaults]
host_key_checking = False
```
And now, we can ping to our VMs to check if the connection is ok, let's ping it:

```shell
$ ansible -i inventory servers -m ping
```

**Explaining:**

-i = path to our inventory file.

servers = name of our group, but it's possible to write the nome of the VM, for example: vm1 or vm2.

-m = module name to execute, in this case the module ping.

If the ping goes well, Ansible will return the following: 

```shell
vm1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
vm2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Playbook

Remember, playbook containing a series of tasks and instructions that are executed on one or more remote hosts. And we have two remote hosts, vm1 and vm2.

To get started, let's make a playbook that will just run one command in vm1, in this case the date command.

Creat a file called playbook.yaml and put the following content:

```yaml
- name: 'Execute a date command on vm1'
  hosts: vm1
  tasks:
    - name: 'Execute a date command'
      command: date

```
Remember, playbooks are yaml files, so, pay attention to the indentation.

**Explaining:**

\- name: name of the playbook

hosts: name of the target vm

tasks: list os tasks

\- name: just a name for the task

command: name of the module to be executed. Its value, in this case, is a command that will be executed.

Save and run:

```shell
$ ansible-playbook -i inventory playbook.yaml
```
**Explaining:**

ansible-playbook: command that runs an Ansible playbook

-i: argument to use an inventory

inventory: path to a inventory file

playbook.yaml: path to a playbook file

You will receive an output like this:
```shell
PLAY [Execute a date command on vm1] *************************************************************

TASK [Gathering Facts] ***************************************************************************
ok: [vm1]

TASK [Execute a date command] ********************************************************************
changed: [vm1]

PLAY RECAP ***************************************************************************************
vm1        : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Well, we execute the date command inside vm1, but is possible the read the output of date command executed inside vm1 directly in ansible ouput, just pass the parameter -v in ansible-playbook command line. To see this let's execute the date command in both VMs, we just need to chance in playbook file the value of key hosts to the name of server group, this way:

```yaml
- name: 'Execute a date command on servers group'
  hosts: servers
  tasks:
    - name: 'Execute a date command'
      command: date

```
And now, let's run the playbook with -v param:
```shell
$ ansible-playbook -i inventory playbook.yaml -v
```

Now you will be able to notice that below the name of the task there will be a log with some information, including the executed command and its return:

```shell
changed: [vm1] => {"changed": true, "cmd": ["date"], ... "stdout": "Thu Mar 16 01:35:18 UTC 2023",...
```

We can create a playbook with many tasks, try to execute this:
```yaml
- name: 'Execute commands on servers'
  hosts: servers
  tasks:
    - name: 'Execute a date command'
      command: date
    - name: 'Execute an echo command'
      shell: echo 'testing'
    - name: 'Execute 3 commands with some pipe'
      shell: echo 'testing another string' | grep 'string' | wc -l
```

What if we want to run a command like sudo? How would we do it? Simple, using the **become** directive. If we want all tasks to be executed with root privileges, just put **become** before tasks, below hosts, for example. But we can also put the **become** together with the task, as in the example below:

```yaml
- name: 'Execute a date command on vm1'
  hosts: servers
  tasks:
    - name: 'Execute a date command'
      command: date
    - name: 'Execute an echo command'
      shell: echo 'testing'
    - name: 'Execute 3 commands with some pipe'
      shell: echo 'testing another string' | grep 'string' | wc -l
    - name: 'Create a file'
      shell: touch /root/myfile.txt
      become: true
```
But to execute this playbook, and the **become** directive really works, you need to use -K in the ansible-playbook, this parameter will ask for the sudo password:
```shell
$ ansible-playbook -i inventory playbook.yaml -v -K
```

Is it possible to run a task with root privilege without having to put a sudo password to Ansible? The answer is yes. But you will need to change the behavior of sudo in the VM.

For this you will have to access the VM and create a file like this:
```shell
$ sudo vim /etc/sudoers.d/ansible_user_nopass
```

And put the following content in this file:
```ini
user ALL=(ALL) NOPASSWD: ALL
```
In place of *user* put the correct user.

And that's it, now it is possible to run the playbook's tasks with become without having to pass the -K parameter, therefore, without having to type the sudo password.

Now, to finish, let's install a package in one VM, for example, in VM1. Just to separate the things, let's create another playbook with the name **playbooks_apt.yaml** with the following content:
```yaml
- name: 'Manage packages'
  hosts: vm1
  become: true
  tasks:
    - name: 'Install VIM'
      apt:
        name: vim
        update-cache: yes
    - name: 'Where is vim?'
      shell: whereis vim

```
After installing vim, the next task does a whereis looking for vim on the system.


**Explaining (just new things):**

apt: this module manages apt packages

name: package's name (in this case, vim)

update-cache: this directive is the same thing as *apt update*

Just run the playbook:

```shell
$ ansible-playbook -i inventory playbook2.yaml -v
```

The output of the task **TASK [Where is vim?]** should look something like this:
```shell
"stdout": "vim: /usr/bin/vim /etc/vim /usr/share/vim", [...]
```
And if is necessary to remove the package, just add the **state** directive with **absent** value, like this:
```yaml
- name: 'Manage packages'
  hosts: vm1
  become: true
  tasks:
    - name: 'Remove VIM'
      apt:
        name: vim
        update-cache: yes
        state: absent
    - name: 'Where is vim?'
      shell: whereis vim
```
And that is the first part of this beginner's Ansible tutorial. When the next part is finished I will update this post and link the new post here.


