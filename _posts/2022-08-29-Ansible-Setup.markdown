---
layout: post
title: "Setup Ansible Controller Node and Remote Servers"
date: 2022-08-29 10:38:00 +0300
categories: Ansible Automation Prometheus Grafana
author: Salim Said Hemed
# pulished: false
---

# 1. Ansible Introduction and Basic Setup

[Ansible](<https://en.wikipedia.org/wiki/Ansible_(software)>) is an Infrastructure as code tool that is used to _provision_, Perform _Configuration Management_ and automate _Application Deployment_.

## i. Characteristics

### a. Idempotency

An operation is idempotent if the result of performing it once is exactly the same as the result of performing it repeatedly without any intervening actions.

For Ansible it means that if an operation is done once and a change is made the next time the operation is performed there will be no effect.

### b. Agentless

Unlike other Configuration management tools Ansible does not require Software (Client Agents) to be installed on the Nodes to be monitored, it just requires OpenSSH on Linux and WinRM on Windows to be configured. As these tools are already implemented on existing servers getting Ansible setup is an easy task.

The Connection is temporary and When Ansible is not managing a node, it does not consume resources on the node because no daemons are run or software installed.

# 2. Architecture

The basic Architecture of Ansible is as shown below:

![Ansible Architecture]({{site.url}}/assets/img/ansible-intro/architecture.png)

As shown above a typical Ansible setup consists of:

**A Controller Node**: This is the node/machine that manages/orchestrates the target machines using the SSH Protocol. This is where Ansible is Installed. It is typically a Linux Machine. Unlike other configuration management tools Ansible does not limit the number of Master Nodes this means we can have more than one Controllers reducing the risk of failure if one of the nodes goes down.

Ansible requires Python to be installed on all managing machines, including pip package manager along with configuration-management software and its dependent packages. Managed network devices require no extra dependencies and are agentless.

**Target Hosts**: These can be any Linux or Windows hosts that can receive instructions via SSH or WinRM and have changes applied to them. As mentioned above Ansible is agentless we therefore do not need to install any packages on the target hosts.

# 3. Setup

## i. The Controller Node

### a. Installation

This section covers the Installation of the Ansible Controller Node, as specified above Ansible is agentless, that is we only need to install it on the Controller node and on the hosts being controlled we just need to have OpenSSH Running.

Ansible is included as part of the Fedora distribution of Linux, owned by Red Hat, and is also available for:

- Red Hat Enterprise Linux,
- CentOS,
- openSUSE,
- SUSE Linux Enterprise,
- Debian,
- Ubuntu,
- Scientific Linux
- Oracle Linux via Extra Packages for Enterprise Linux (EPEL)

Also note that a Ansible is written in python it can also be installed as a pip module in ny environment that can run Python

This section will cover installation on Ubuntu and CentOS.

#### **Installation on Ubuntu**

Ansible is not available on the base Ubuntu repos, therefore it has to be installed by adding a [PPA](https://launchpad.net/~ansible/+archive/ubuntu/ansible):

1. Update your apt repositories/sources:

   `sudo apt update`

2. Install `software-properties-commom` which will _provide an abstraction of the used apt repositories. It allows you to easily manage your distribution and independent software vendor software sources_ see [What is software-properties-common](https://askubuntu.com/questions/1000118/what-is-software-properties-common).

   `sudo apt install software-properties-common`

3. Add Ansible PPA Repository and update:

   `sudo add-apt-repository --yes --update ppa:ansible/ansible`

4. Install Ansible:

   `sudo apt install ansible`

#### **Installation on Fedora/CentOS**

Ansible is already included in the Fedora Repos, so to install it you Simply Run:

`sudo dnf install ansible`

On CentOS it is part of the epel packages, so these repos must be enabled:

```
sudo yum install epel-release
sudo yum install ansible
```

#### **Other Platforms**

Ansible Cannot run on Windows, However I have had Success running it on WSL/WSL2 Ubuntu, you can follow the [instructions on setting up WSL/WSL2](https://docs.microsoft.com/en-us/windows/wsl/install) and then install Ansible following the instructions for [Ubuntu](#installation-on-ubuntu) above. See [Can Ansible run on Windows?](https://docs.ansible.com/ansible/latest/user_guide/windows_faq.html#windows-faq-ansible)

Ansible can also be installed on any paltform that has pip and python enabled. see [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

### b. SSH Setup on the Controller Node

Once Ansible is installed on the controller node, we need to facilitate the ssh connection between the Controller and the target machines. Normally any user can be used to send the SSH Commands remotely, however i prefer to use a specially created user for this purpose, the user is aptly named `ansible`, to ensure uniformity a similiar user will be created on all the target nodes the only difference is that the user on the remote machines will have sudo privileges and will be allowed to run sudo commands without being prompted for a password.

To create the ansible user:

```bash
sudo useradd ansible
```

Although ansible can authenticate with the remote machines using Password authentication, it is highly recommended to use Key Based authentication, to enable this from the controller Node to all the target nodes follow the steps below:

1. To generate ssh keys for the user:

   `sudo su - ansible`
   `ssh-keygen` and follow instructions

This step is only done once on the controller Node.

2. To copy the key to the remote servers(Repeat this for all the remote servers),the ansible user will be created on the remote servers in the next section:

   `ssh-copy-id ansible@<REMOTE_IP>`

SSH Key for the Ansible User is copied from the Controlling Node to the Remote Servers to facilitate SSH Key Based Authentication.

## i. The Target Hosts

For the remote hosts the only action that needs to be done is to create a user with sudoers permissions named ansible for uniformity's sake and allow the user to run elevated commands without being prompted for a password. To achieve this follow the steps below:

1. Create `ansible` user:

   `sudo useradd ansible`

2. Ensure the user is a member of the sudoers group:
   `usermod -aG sudo ansible` or `usermod -aG wheel ansible` for CentOS/Fedora/RedHat systems.

3. Ensure that the user can execute sudo commands without being prompted for a password

   - Open the `sudoers` file : `visudo`

   - Add the Following Entry: `ansible ALL=(ALL:ALL) NOPASSWD: ALL`

   - Save Changes made to the sudoers File

4. The server should now be ready to receive remote commands via ssh using Ansible

Ansible should now be setup on both the controller and remote hosts.

In future articles we will use ansible to provision Applications and Software stacks.

For More Ansible Goodness and references see [Ansible Documentation](https://docs.ansible.com/)

- [Getting Started with Ansible](https://docs.ansible.com/ansible-core/devel/getting_started/index.html)
- [Installation Guide](https://docs.ansible.com/ansible-core/devel/installation_guide/index.html)
- [Ansible Inventories](https://docs.ansible.com/ansible-core/devel/inventory_guide/index.html)
- [Using Ansible command line tools](https://docs.ansible.com/ansible-core/devel/command_guide/index.html#using-ansible-command-line-tools)
- [Ansible Playbooks](https://docs.ansible.com/ansible-core/devel/playbook_guide/index.html)
- [Ansible Tips and Tricks](https://docs.ansible.com/ansible-core/devel/tips_tricks/index.html)
