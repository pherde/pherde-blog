---
title: "Ansible: A Beginner's Tutorial - Part 1."
date: 2023-02-14T21:29:10-03:00
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


For our tests we initially need two virtual machines with an installed system, I chose to use **QEMU** as my VM and install the **Alpine Linux** system in both VMs. But everything we do here can be done with Virtbualbox and Ubuntu, for example. The important thing is to install the system and be able to access it via ssh. So, our scenario will be:

| Host (with Ansible) | VMs |
| --------------------|-----|
| Ubuntu 20.04        | Alpine Linux 1 |
|                     | Alpine Linux 2 |

I recommend using **virt-manager** on Ubuntu, it is the simplest way to work with QEMU in my opinion.

Don't forget to install Ansible in the host machine, in my case in ubuntu:

```shell
$sudo apt install ansible
```

### Inventory

Remember, the inventory contains the list of hosts that Ansible will access to make its magic.

With our VMs running we are going to write our inventory, so for this we will create a file called `inventory` with the following content:

```ini
vm1 ansible_host=192.168.122.90 ansible_connection=ssh ansible_user=fernando ansible_ssh_pass=mypass!12
vm2 ansible_host=192.168.122.91 ansible_connection=ssh ansible_user=fernando ansible_ssh_pass=mypass!12

[serveralpine]
vm1
vm2
```

**Explaining:**

vm1, vm2 = name, like a variable, for our target host.

ansible_host = address of our target.

ansible_connection = connection type.

ansible_user = name of the user who will access the machine.

ansible_ssh_pass = user password.

[serveralpine] = group name, we can add as many hosts as we want into the group.

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
$ansible -i inventory serveralpine -m ping
```

**Explaining:**

-i = path to our inventory file.

*serveralpine* = name of our group, but it's possible to write the nome of the VM, for example: vm1 or vm2.

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


