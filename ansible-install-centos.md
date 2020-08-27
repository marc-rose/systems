# Procedure - Install Ansible

## Procedure Purpose

This document demonstrates the steps to install Ansible manually in Linux

## Procedure Scope

This procedure applies to and is tested with:
- CentOS 8
- Ansible 2.9
- Python 3.8

### Required Packages:
   | Package             | Purpose                                     |
   |---------------------|:-------------------------------------------:|
   | epel-release        | repository containing extra packages        |
   | python38            | latest Python                               |
   | ansible             |                                             |
   | python3-argcomplete | provides shell completion as of ansible 2.9 |


#### Documents used as reference:
-	https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
- https://www.liquidweb.com/kb/how-to-install-and-configure-ansible/

#### Sections:
-	Install Ansible Environment

## Procedure Statement

### Install Ansible Environment
1. Join to server via SSH and issue the install packages:
   1. Install EPEL repository to ensure all packages are listed
   
      ```
      $ sudo yum install epel-release
      ```
   
   1. Install remaining packages
   
      ```
      $ sudo yum install python38 ansible python3-argcomplete
      ```

1. Activate python3-argcomplete globally

      ```
      $ sudo activate-global-python-argcomplete
      ```

1. Ensure `python` uses intended version:
   
   ```
   $ python -V
   $ sudo alternatives --config python
   $ python -V
   ```

1. [Optional] Create user that will run Ansible playbooks that is not sudo

   ```
   $ sudo useradd ansibleuser
   ```
