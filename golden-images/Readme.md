# Procedure - Windows 10 Templates

## Procedure Purpose

This document demonstrates the steps to create a Windows 10 Workstation VM Golden Image in vCenter

## Procedure Scope

Servers and workstations deployed from VM Templates using a Guest OS Customization (GOSC) Specification require an image be created with certain requirements for each OS. Following are the minimum requirements for GOSC.
Template can be targeted with an infrastructure provisioning tool such as Terraform to scale the process of deploying entire environments.

#### Common requirements:
-	Generic local administrator and/or root login credentials

#### Windows Requirements:
- Windows Features
  - Telnet Client
- Simple Network Management Protocol (SNMP) - for SNMP monitoring
- Applications 
  - VMWare Tools
  - Google Chrome
  - Adobe Reader
- Utilities - C:\Utilities\ 
  - treesize
  - tcpview
  - PuTTY
  - winscp
  - SysInternals utilities
- Modifications to VDI Image Optimization files 
  - AppxPackages.json - List of Applications removed such as Xbox, Calculator, OneDrive 
    - Remove from list and keep installed - Microsoft.MSPaint, Microsoft.WindowsCalculator
  - AutoLoggers.Json - List of logging and tracing providers to disable such as Cellcore, WiFiSession 
    - No change
  - ScheduledTasks.json - List of Tasks removed such as ScheduledDefrag and Microsoft-Windows-DiskDiagnosticDataCollector 
    - No change
  - Services.json - List of Services disabled such as VSS, Power, XblGameSave 
    - Change to **"VDIState": "Enabled"** for services: UsoSvc, VSS
  - DefaultUserSettings - Default registry keys to optimize UI 
    - No change

#### Not necessary:
- sysprep before converting to template: sysprep is automatically performed when deploying from GOSC profile
- Joining to AD Domain: Computer will be added automatically via deployment tool or GOSC profile

#### Documents used as reference:
-	https://thevirtualhorizon.com/2017/03/13/my-windows-10-template-build-process/
-	https://notesfrommwhite.net/2016/12/11/how-to-build-a-windows-2016-vmware-template/
-	https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/rds_vdi-recommendations-1909
-	https://github.com/The-Virtual-Desktop-Team/Virtual-Desktop-Optimization-Tool#full-instructions-for-windows-10-2004-or-windows-10-1909
    - The Virtual Desktop Optimization Tool was created by Windows experts working for Microsoft Premier Services that turned a lengthy Windows 10 VDI Image Optimization document into a set of scripts

#### Sections:
-	Configure Virtual Machine in vCenter
-	Install Operating System
-	Perform OS tweaks and customizations
-	Cleaning the virtual machine configuration

## Procedure Statement

### Configure Virtual Machine in vCenter
Follow Procedure - vCenter - Create VM from ISO document with the following customizations:
-	Select "Microsoft Windows 10 (64-bit)" as guest OS for the virtual machine.
-	SW_DVD9_Win_Pro_10_1903_64BIT_English_Pro_Ent_EDU_N_MLF_X22-02890.ISO is most recent ISO used to test this document

Install the Operating System
1. Launch the Windows Setup. Click **Next**, then **Install now**
1. Select the correct version of Windows 10. Click **Next**. Accept the license terms, click **Next**.
1. Choose **Custom installation** and **Drive 0 Unallocated Space**. Click **Next**.
1. Installer will restart automatically and reboot after Getting ready.
1. Choose the region **United States** and keyboard layout **US**.
1. When prompted to Sign in with Microsoft, choose **Domain join** instead. Enter a temporary local user and leave the password field blank.
1. Opt out of and decline optional tracking features
1. The installer will finish customizing Windows before loading the UI with the temporary administrator account.
1. Verify the workstation has obtained an IP via DHCP or assign a static IP
1. Check for and install Windows Updates. This may take several rounds of installs and reboots.

   **NOTE:** after optimization steps with Virtual Desktop Optimization Tool, Windows Updates may be paused until manually enabled
   
1. Install VMware Tools
   1. Click **Install VMWare Tools...** from vCenter. Mount CD/DVD drive when prompted
   1. From the OS of the template open the mounted drive and run **setup64.exe** as administrator. Perform install with default settings and reboot when prompted
1. Disable Windows Defender Firewall for the active network profile the machine is using (Public by default).

   **NOTE:** This may be required to access the machine remotely
   
   - Alternatively, you can allow certain traffic with custom firewall rules
1. Install additional applications and standalone utilities as required for build

#### Perform OS tweaks and customizations

1. Rename the VM in the OS to match the template name
1. Open **System Properties > Advanced tab > Performance > Settings > Visual Effects tab: Adjust for best performance**
1. Add Windows Features: run PowerShell as Administrator. Run commands to install
   -  Telnet Client 
   
      `Enable-WindowsOptionalFeature -Online -FeatureName TelnetClient`
   
1. Install the SNMP Service: open PowerShell as Administrator. Run commands to install and verify

   **NOTE:** A reboot is required before an SNMP community string can be configured to the service. Group Policy should be used to push the string if necessary
   
   `Add-WindowsCapability -Online -Name "SNMP.Client~~~~0.0.1.0"`
   `Get-WindowsCapability -Online -Name "SNMP*"`
      
1. Uninstall any remaining Programs that are unneeded.
1. Check for and install any remaining updates to the machine and reboot.
1. Run the Virtual-Desktop-Optimization-Tool

   **NOTE:** This will cause Windows Updates to stop functioning until steps are made to re-enable 
   
1. Browse to https://github.com/The-Virtual-Desktop-Team/ and find the Virtual-Desktop-Optimization-Tool project. The readme contains the steps and the folder contain all the needed files.

   **NOTE:** this document applies to Windows 10 Build 1909, the latest build Windows Updates would apply when last updating this document. Change any references for builds (2004, 1909) to the installed build
   
   1. Copy the files as instructed to a folder **C:\Optimize** on the VDI template machine
   1. In the Properties of Win10_VirtualDesktop_Optimize.ps1 check Unblock and click OK. We can now allow running of this script without modifying the ExecutionPolicy (RemoteSigned)
   1. Make any necessary modifications to the following files to customize further: 
      - AppxPackages.json
      - AutoLoggers.Json
      - ScheduledTasks.json
      - Services.json
      - DefaultUserSettings
1. Open the Win10_VirtualDesktop_Optimize.ps1 file and Save.
1. Run the following commands (modified for build 1909) in PowerShell. The process should take a few minutes and suggest a reboot.
   
   **NOTE:** Always run PowerShell as Administrator
   
   `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned`
   `C:\Optimize\Win10_VirtualDesktop_Optimize.ps1 -WindowsVersion 1909 -Verbose`
   
1. Open System Properties > Advanced tab > Performance > Settings > Visual Effects tab: Adjust for best performance
1. Uninstall any programs that are unneeded.

#### Cleaning the virtual machine configuration and converting to template
1. Ensure the appropriate local accounts exist and unnecessary accounts are removed
1. Enable Windows Updates and allow the machine to contact Windows Update
   1. Open the Local Group Policy Editor and enable **Select when Quality Updates are received**
   1. Ensure **Update Orchestrator Service** (UsoSvc) is set to **Automatic (Delayed Start)**
   1. Run any updates to the latest version/build.
1. Run Virtual-Desktop-Optimization-Tool to re-apply any base optimization settings

   `C:\Optimize\Win10_VirtualDesktop_Optimize.ps1 -WindowsVersion 1909 -Verbose` 

4. Run cleanup for old OS update files

   `Dism.exe /online /Cleanup-Image /StartComponentCleanup`
   
5. Clear all logs

   `Clear-EventLog -LogName (Get-EventLog -List).log`
   
6. Shut down PC
7. Disconnect any mounted drives for the VM
8. Add any notes to the VM before converting to template. Make sure to add a dated line summarizing the changes made to the template.
9. Convert VM to template

#### VM Maintenance
Regular maintenance should be done to ensure templates are secure and up-to-date
1. Convert template to VM
2. Make necessary changes 
   - Manage applications
   - Run Windows Updates
3. Clean the virtual machine configuration and convert to template (see section in document)
