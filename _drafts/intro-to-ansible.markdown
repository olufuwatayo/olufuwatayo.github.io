---
title: Intro to Ansible
date: 2020-02-29 22:52:00 Z
---

Have you ever had to manage more than a single Linux server? Having to ssh into the severs and typing individual commands on the terminal. What happens if you have to manage 20 or 30 servers? What happens if you make a mistake and you need to start all over?

How many hours would it take?
What of the documentation and dependencies?
These are the challenges you would face if you don't automate the process of managing your configuration. Luckily there is a concept that helps us achieve this and it's called Config Management.

[Configuration management (CM)](https://en.wikipedia.org/wiki/Configuration_management) is a systems engineering process for establishing and maintaining consistency of a product's performance, functional, and physical attributes with its requirements, design, and operational information throughout its life. **ELI5**: what this means is that you have your software installations, your configurations, and your updates defined as a code that is re-usable, understandable and version controlled.

If you need to make changes to your configurations you can just update your iac codes and run it on your infrastructure without fear of it breaking your systems.

This is going to be a basic article to get you started with Ansible and is intended for someone that has never used ansible before.

Benefit of using configuration management include:
Speed
Automation
Version Control
Team collaboration
Repeatable


Ansible: Ansible is an open-source software provisioning, configuration management, and application-deployment tool. It runs on many Unix-like systems, and can configure both Unix-like systems as well as Microsoft Windows.

Components of ansible core

**Control node**:
Any machine that has Ansible installed. You can run ansible ad-hoc commands and Ansible playbooks from your control node. Any computer that has Python installed can be a control node - laptops, shared desktops, and servers can all run Ansible. However, you cannot use a Windows machine as a control node.

**Ansible slaves / host**
The resources you manage with Ansible. Ansible is not installed on managed nodes but you need python installed for unix hosts and PowerShell 3.0 or newer and at least .NET 4.0 to be installed on the Windows host. Also, A WinRM listener should be created and activated for windows node.

**Inventory**
A list of ansible hosts. An inventory file is also sometimes called a “hostfile”. Your inventory can specify information like IP address for each managed node. An inventory can also organize managed nodes, creating and nesting groups for easier scaling.

**Modules**
The units of code Ansible executes. Each module has a particular use, from administering users on a specific type of database to managing VLAN interfaces on a specific type of network device. You can invoke a single module with a task, or invoke several different modules in a playbook.

**Tasks**
The units of action in Ansible. You can execute a single task once with an ad-hoc command.

**Playbooks**
Ordered lists of tasks, saved so you can run those tasks in that order repeatedly. Playbooks can include variables as well as tasks. Playbooks are written in YAML and are easy to read, write, share and understand.

#installation we would be covering how to install ansible on Unix, for windows you can see this section


Ensure you have python installed (most MacOs already have it pre-installed) else you might have to install it on your operating system.
To install Ansible on MacOS type `brew install ansible` on your terminal.

Setting up a simple web-server with ansible: 

To do that we can need to create three files.

**main.yml** this would be our entry point for our ansible-playbook

Copy this to the main.yml.
{% highlight yaml %}
---

- name: Install Lamp Server
  gather_facts: false
  become: True
  become_user: root
  become_method: sudo
  hosts: all
  tasks:
    - name: Install Lamp Server
      apt:
        name: nginx
        state: present
{% endhighlight %}


**ansible.cfg** this is the file that defines the configs that allow ansible to interact with our host and other settings needed.

[defaults]
host_key_checking = False #You can disable this so ansible does not ask for host key here
remote_user = ubuntu #Your ssh user should be here
private_key_file = ~/.ssh/id_rsa #link to your private key here



**hosts.ini** we would store our ansible host file here so ansible can connect to them.
Just store your ip address in here and ensure that you can connect to it with your ssh key.


