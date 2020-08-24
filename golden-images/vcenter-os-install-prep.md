# Procedure - Prepare to Install Server OS in vCenter

## Procedure Purpose

This document demonstrates the steps to begin an install of an OS in vCenter. Subsequent steps will be covered in corresponding documents specific to the OS required

## Procedure Scope

Depending on the OS requirements, some settings may be altered:

vSphere compatibility: OS type

Important settings for VM deployments:

Network
- vCenter Adapter Type: VMXNET 3
- DHCP set to Automatic, common settings (IP, DNS, namespace) should not be statically assigned

BIOS
- Advanced > I/O Configuration
- Disable Serial ports, Parallel port, Floppy controller

Documents used as reference:
- https://blog.inkubate.io/create-a-centos-7-terraform-template-for-vmware-vsphere/


## Procedure Statement

### Configure Virtual Machine in vCenter
1. Create a new virtual machine.
1. Choose the name for your virtual machine template.
      - This should match the of the machine to be created.
      - To minimize NETBIOS name issues, keep machine names to 15 characters or less.
1. Select a temporary compute resource for the virtual machine.
1. Select a datastore for the virtual machine. The first datastore is generally designated for Templates and ISOs
1. Select the vSphere compatibility for the virtual machine.
1. Select the appropriate guest OS for the virtual machine.
1. Configure the appropriate hardware settings.
   1. Virtual Hardware tab: Keep resources to the minimal requirements so our clones are lean.
      - CPU
      - Memory
      - Hard Disk
      - Network: Set to a staging VLAN, ideally one with DHCP configured. This can streamline the template creation process.
        **NOTE:** ensure VMXNET 3 is selected
      - CD/DVD Drive: Add the appropriate ISO to the CD/DVD drive of the virtual machine. ISOs are usually located in the ISO folder on the first Datastore of the cluster. Ensure Connect At Power On is checked.
   1. VM Options tab
      - Ensure Force BIOS setup is checked. We will be modifying the BIOS in a later step.
      - Validate the creation of the virtual machine. Click Finish.
1. Power on the virtual machine and Launch the vSphere web console.
   1. Under **Advanced > I/O Device Configuration** disable the Serial ports, Parallel ports, and Floppy disk controller.
   1. Press F10 to save and exit.
1. Proceed with OS install
