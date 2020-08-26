# Procedure - CentOS Template

## Procedure Purpose

This document demonstrates the steps to create a CentOS VM Golden Image in vCenter

## Procedure Scope

Servers and workstations deployed from VM Templates using a Guest OS Customization (GOSC) Specification require an image be created with certain requirements for each OS. Following are the minimum requirements for GOSC.
Template can be targeted with an infrastructure provisioning tool such as Terraform to scale the process of deploying entire environments.

#### Common requirements:
-	Generic local administrator and/or root login credentials

#### Linux Requirements:
- Common packages to be installed and regularly updated:
  | Package Name   | Purpose                  |
  |----------------|:------------------------:|
  | perl           | needed for vmtools       |
  | open-vm-tools  | open version of vm-tools |
  | firewalld      | firewall service         |
  | net-snmp       | for snmp monitoring      |
  | net-snmp-utils | for snmp monitoring      |
  | yum-utils      | for cleaning yum         |
- Firewall/security settings I prefer for a Linux beginner
   - use firewalld for software firewall
   - configure SELinux to **Permissive** mode. SELinux will still log traffic that would have been blocked if in **Enforced** mode

#### Documents used as reference:
-	https://blog.inkubate.io/create-a-centos-7-terraform-template-for-vmware-vsphere/
- https://www.tecmint.com/firewalld-rules-for-centos-7/
- https://community.webcore.cloud/tutorials/linux_specific_articles/how_to_extend_partition_with_unallocated_space_cen/
- https://lonesysadmin.net/2013/03/26/preparing-linux-template-vms/
- https://www.logicmonitor.com/support/monitoring/os-virtualization/snmp-ntp-configuration-for-linux-devices
- https://www.tecmint.com/firewalld-rules-for-centos-7/

#### Sections:
-	Configure Virtual Machine in vCenter
-	Install Operating System
-	Perform OS tweaks and customizations
-	Cleaning the virtual machine configuration

## Procedure Statement

### Configure Virtual Machine in vCenter
Follow vcenter-os-install-prep.md document with the following customizations:
-	CentOS Minimal Installer is sufficient and will help reduce template/server size.
-	Select "CentOS 7 (64-bit)" or "CentOS 8 (64-bit)" as guest OS for the virtual machine.

### Install the Operating System
1. Launch the CentOS installer.
1. Select the language of the installer. Click **Continue**.
1. Select the installation disk. Click **Done**.
1. Configure the network card of the virtual machine with a temporary (static or DHCP) configuration. The network settings will be removed in a later step. In this example we are using DHCP.
   1. Give the CentOS machine a temporary hostname. Click **Apply**.
   1. Enable the interface to obtain an IP address. Click **Done**.
1. Click on **Begin Installation**. The installation will happen the background and can take several minutes.

   1. Configure the root password. Make sure this meets complexity requirements, and is documented in an appropriate location.
   1. Configure an administrator user. Make sure this is documented in an appropriate location. Click **Done**.

1. When installation is complete, reboot the virtual machine. You may need to click **Finish configuration** before **Reboot** is available.
1. Connect to the VM by one of the following methods:
   - SSH to the new CentOS virtual machine via CLI or SSH client with the local administrator account. You should be able to connect via hostname if DNS/WINS/NETBIOS is functioning. The IP can be found by using the command `ip addr`
   - vSphere console. This can cause difficulty when attempting to copy and paste text.
1. Upgrade the installed CentOS packages

   `$ sudo yum upgrade [-y]`

1. Install required packages listed in Procedure Scope (open-vm-tools, perl, etc). You can optionally add all packages in one command

   `$ sudo yum install <package-name> [<package-name-2> <package-name-3> ...] [-y]`

1. [Optional] Configure SNMP for monitoring.

   **NOTE:** **net-snmp** and **net-snmp-utils** are required to be installed. This example is for SNMPv2

   1. Move the default config file and keep it as backup.
   
   `$ sudo mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.original`
   
   1. Define the SNMP community for standard server monitoring. This string should be well-documented for our environment. The following command creates a simple config for SNMP monitoring, and can be modified to change the string or IPs allowed to reach this server.
   
   **NOTE:** customize command by substituting values for `<server-snmp-community-string>` and `<ip-in-cidr-format>`
   
   `$ sudo sh -c "echo 'rocommunity <server-snmp-community-string> <ip-in-cidr-format>' > /etc/snmp/snmpd.conf"`

   1. Start the daemon, and set it to start on server boot.

   `$ systemctl restart snmpd.service`
   
   `$ systemctl enable snmpd.service`
   
   1. Allow inbound traffic through the software firewall (firewalld) toward SNMP ports. The --permanent switch will persist after reboot
   
   `$ sudo firewall-cmd --permanent --add-service=snmp`
   
   `$ sudo firewall-cmd --reload`
   
   `$ sudo firewall-cmd --permanent --list-all`

1. Set SELinux mode to Permissive. SELinux will still log what would have been blocked in case we want to enforce later
   1. Check status of SELinux. SELinux is enabled and **enforcing** by default.
   
   `$ sestatus`
   
   1. Edit **/etc/selinux/config** file and change/set the following line: `SELINUX=permissive`
   
   **NOTE:** The following command automatically replaces the line. The status will not change until after a reboot or when temporarily changed to **permissive** with `sudo setenforce 0`
   
   `$ sudo sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config`

1. Reboot the virtual machine.

   `$ sudo reboot`


#### Cleaning the virtual machine configuration and converting to template

The steps outlined in this section are meant to delete temp files and logs before converting to template.

1. SSH to the new CentOS virtual machine via CLI or SSH client.
1. Switch user access to root level. You will need to enter the root secret. This step will streamline the deletion and accessing of temp folders.

   **NOTE:** the prompt symbol $ will change to a # 

   `$ su`

1. Stop logging services.

   `# /sbin/service rsyslog stop`
   
   `# /sbin/service auditd stop`

1. Remove old kernels.

   `# /bin/package-cleanup --oldkernels --count=1`

1. Clean out yum. Yum caches install files and can consume hundreds of MBs of disk

   `# /usr/bin/yum clean all`

1. Force the logs to rotate & remove old logs we don’t need. You may receive an error: /bin/rm: cannot remove â€˜\226fâ€™: No such file or directory

   `# /usr/sbin/logrotate –f /etc/logrotate.conf`
   
   `# /bin/rm –f /var/log/*-???????? /var/log/*.gz`
   
   `# /bin/rm -f /var/log/dmesg.old`
   
   `# /bin/rm -rf /var/log/anaconda`

1. Truncate the audit logs (and other logs we want to keep placeholders for).

   `# /bin/cat /dev/null > /var/log/audit/audit.log`
   
   `# /bin/cat /dev/null > /var/log/wtmp`
   
   `# /bin/cat /dev/null > /var/log/lastlog`
   
   `# /bin/cat /dev/null > /var/log/grubby`

1. Clean /tmp out. Normally we don't like to empty /tmp but for template images it will be fine.

   `# /bin/rm –rf /tmp/*`
   
   `# /bin/rm –rf /var/tmp/*`

1. Remove the SSH host keys. This is a good security step to ensure multiple servers don't have the same SSH host keys

   `# /bin/rm –f /etc/ssh/*key*`

1. Remove the root user’s shell history.

   `# /bin/rm -f ~root/.bash_history`
   
   `# unset HISTFILE`

1. Remove the root user’s SSH history & other cruft. Optionally, you might choose to just remove ~root/.ssh/known_hosts if you have SSH keys you want to keep around.

   `# /bin/rm -rf ~root/.ssh/`
   
   `# /bin/rm -f ~root/anaconda-ks.cfg`

1. Shut down before converting to template

   `# shutdown now`

1. Add any notes to the VM before converting to template. Make sure to add a dated line summarizing the changes made to the template.

1. Convert VM to template


#### VM Maintenance

Regular maintenance should be done to ensure templates are secure and up-to-date

1. Convert to VM
1. Make necessary changes
   - Manage packages
      - Update all yum packages
      - Install and remove packages
   - Manage firewall rules
1. Clean the virtual machine configuration and convert to template (see section in document)

##### Managing packages

- List all installed yum packages:

  `$ sudo yum list installed | less`
  
- List Yum packages with pending updates:

  `$ sudo yum list updates`

  `$ sudo yum check-update

- Install package with Yum:

  `$ sudo yum install <package-name>`

- Install multiple packages with Yum:

  `$ sudo yum install <package-name> [<package-name-2> <package-name-3> ...] [-y]`
  
- Update individual yum package:

  `$ sudo yum update <package-name>`
  
- Update all yum packages:

  `$ sudo yum update`
  
- Remove yum package:

  `$ sudo yum remove <package-name>`

##### Firewall rules

Firewall rules are controlled by firewalld and will need to be managed on the command line any time a new method of connecting to the server is required. e.g. web traffic to a web server

Ports can be opened specifically by port via a specific TCP or UDP number (80/tcp or 161/udp) or by service with a name (http or snmp). Unless specified otherwise, the change will apply to the zone public

- Adding ports in firewalld

  `firewall-cmd --permanent --add-port=80/tcp`

  `firewall-cmd --permanent --add-port=443/tcp`

  `firewall-cmd --permanent --add-port=123/udp`

- Removing ports in firewalld

  `firewall-cmd --permanent --remove-port=80/tcp`

- Confirming ports in firewalld

  `firewall-cmd --permanent --list-ports`

- Adding services in firewalld

  `firewall-cmd --zone=public --add-service=http`

  `firewall-cmd --zone=public --add-service=https`

  `firewall-cmd --zone=public --add-service=ntp`

- Confirming services in firewalld

  `firewall-cmd --permanent --list-services`
