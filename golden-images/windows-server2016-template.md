# Procedure - Windows Server 2016 Template

## Procedure Purpose

This document demonstrates the steps to create and maintain a Windows Server 2016 VM Golden Image in vCenter

## Procedure Scope
Servers and workstations deployed from VM Templates using a Guest OS Customization (GOSC) Specification require an image be created with certain requirements for each OS. Following are the minimum requirements for GOSC.
Template can be targeted with an infrastructure provisioning tool such as Terraform to scale the process of deploying entire environments.

#### Common requirements:
-	Generic local administrator and/or root login credentials

#### Windows Requirements:

- Windows Features
  - Simple Network Management Protocol (SNMP) - for SNMP monitoring
  - Telnet Client
- Applications
  - VMWare Tools
  - Dameware
  - Google Chrome
  - Adobe Reader
- Utilities - C:\Utilities\
  - treesize
  - tcpview
  - PuTTY
  - winscp
  - SysInternals utilities
  - VMWare OS Optimization Tool

Modifications to VMWare OS Optimization Tool WindowsServer 2016 - LoginVSI.com - VDILIKEAPRO template

**NOTE:** ensure changes are documented or store the template so we can follow every time we customize a golden image

- Remove the following Steps:
  - Apply HKLM Settings
    - Windows Update - Disable
  - Disabled Services
    - Windows Update
- Steps to consider in future template changes:
  - Removed Services
    - Volume Shadow Copy Service
    - Windows Backup
    - Windows Firewall
  - Removed Features
    - System Restore
    - Firewall (All Profiles)

#### Not necessary:
- sysprep before converting to template: sysprep is automatically performed when deploying from GOSC
- Joining to AD Domain: Computer will be added automatically via GOSC

#### Links used as reference:

- https://notesfrommwhite.net/2016/12/11/how-to-build-a-windows-2016-vmware-template/
- https://flings.vmware.com/vmware-os-optimization-tool

#### Sections:
- Configure Virtual Machine in vCenter
- Install Operating System
- Perform OS tweaks and customizations
- Cleaning the virtual machine configuration


## Procedure Statement

### Configure Virtual Machine in vCenter
Follow vcenter-os-install-prep.md document with the following customizations:
- Select "Microsoft Windows Server 2016 or later (64-bit)" as guest OS for the virtual machine.
- SW_DVD9_Win_Svr_STD_Core_and_DataCtr_Core_2016_64Bit_English_-3_MLF_X21-30350.ISO is the most recent ISO used to test this document

### Install the Operating System
1. Launch the Windows Setup. Click Next, then Install now
1. Select the correct version of Windows Server 2016. Click Next. Accept the license terms, click Next. 
1. Choose Custom installation and Drive 0 Unallocated Space. Click Next.
1. Installer will restart automatically after install and reboot again after Getting ready.
1. Enter a password for the local Administrator account that meets Microsoft's default password complexity requirements. Click Finish
1. Log in with the local Administrator account.
1. Install VMware Tools
   **NOTE:** This may be required to install drivers and establish network connectivity
   1. Click Install VMWare Tools... from vCenter. Mount CD/DVD drive when prompted
   1. From the OS of the template open the mounted drive and run setup64.exe as administrator. Perform install with default settings and reboot when prompted

### Perform OS tweaks and customizations
1. Disable Windows Firewall for Private networks and Guest or public networks. This will allow remote connectivity needed for later steps.
1. Run Windows Updates and reboot as needed until VM is up to date.
1. Install [VMWare OS Optimization Tool](https://flings.vmware.com/vmware-os-optimization-tool) on the VM
   1. Download from site and transfer to VM in C:/
   1. Run the executable to run the tool
1. Download and Customize OSOT Templates
   1. Navigate to the **Public Templates** tab and download the **WindowsServer 2016 - LoginVSI.com - VDILIKEAPRO** template
   1. Navigate to the My Templates tab and select the downloaded template. Click Copy and Edit button to save a copy of the template into MyTemplates folder
   1. Modify the new template to fit the needs of our environment. Use the Search field to find the specific settings and **Remove**. Save template
1. Analyze and Optimize the VM with the VMWare OS Optimization Tool
   1. Ensure the appropriate template is selected and click the **Analyze** button to review optimization items
   1. Review settings that will be changed to see the specific keys that will be affected.
   1. Some steps have been found to not process correctly during Optimization. The step(s) may time out and require additional troubleshooting.
   
      **NOTE:** Steps for disabling Services may say FAILED because the service is not installed
      - Troubleshooting steps:
        - Uncheck or modify the step and **Optimize** again.
        - Reboot the VM before attempting again with all steps checked
      - Known steps that can hang:
        - **Features - Windows Media Player**
        - **Features - WCF-Services**
        - **Features - Xps-Foundation**
   1. Click Optimize button to apply changes
   1. Optimization may take multiple attempts and OS reboots until settings are applied correctly.
1. Rename the VM in the OS to match the template name.
1. Add Windows Features: run PowerShell as Administrator. Run commands to install
   - **Telnet Client**
   ```
   Enable-WindowsOptionalFeature -Online -FeatureName TelnetClient
   ```
   - **SNMP Service**
   ```
   Install-WindowsFeature -Name 'SNMP-Service','RSAT-SNMP'
   ```

1. Install additional applications and standalone utilities as required for build
1. Check one last time for any Windows Updates to be installed.
1. Clean the virtual machine configuration and convert to template (see section in document)

### Maintaining VM Operating System
Regular maintenance should be done to ensure templates are secure and up-to-date
1. Convert template to VM
1. Take a snapshot before making any changes in case you need to roll back
1. Make necessary changes to VM
   - Manage applications
   - Run Windows Updates
1. Make notes in vCenter of any recent changes to the VM
1. Clean the virtual machine configuration and convert to template (see section in document)

### Cleaning the virtual machine configuration and converting to template
1. Use the VMWare OS Optimization Tool (OSOT) to re-apply settings and clean up VM

   **NOTE:** We do not need to **Generalize** since sysprep will be performed during Guest OS Customization Specification 

   1. Analyze and Optimize the VM with the VMWare OS Optimization Tool from the **Optimize** tab
      1. Ensure the appropriate template is selected and click the Analyze button to review optimization items
      1. Review settings that will be changed to see the specific keys that will be affected.
      1. Click **Optimize** button to apply changes
   1. In the Finalize tab, Review settings and click **Execute**. This process will take several minutes and use many tools.
1. Shut down the VM and customize any Virtual Hardware components for the VM.
   1. Set any resources to the minimums required for templates in our environment
      - CPU
      - Memory
   1. Disconnect all CD/DVD drives.
1. Remove any snapshots for the VM
1. Convert to Template
