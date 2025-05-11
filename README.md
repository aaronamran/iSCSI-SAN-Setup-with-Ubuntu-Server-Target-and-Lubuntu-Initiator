# iSCSI-SAN-Setup-with-Ubuntu-Server-Target-and-Lubuntu-Initiator

This write-up documents a practical SAN-like storage setup project using VirtualBox to simulate Storage Area Network (SAN). A Ubuntu Server VM is configured as the iSCSI target VM, while Lubuntu Client VM is configured as the iSCSI initiator VM. 

1. [Configuration of VMs in VirtualBox](#configuration-of-vms-in-virtualbox)
2. [iSCSI Target Setup on Ubuntu Server VM](#iscsi-target-setup-on-ubuntu-server-vm)
3. [iSCSI Initiator Setup on Lubuntu Client VM](#iscsi-initiator-setup-on-lubuntu-client-vm)
4. [Testing of iSCSI Automation]()


## Configuration of VMs in VirtualBox

- Download the Ubuntu Server ISO image suitable for VirtualBox from [https://ubuntu.com/download/server](https://ubuntu.com/download/server). As shown in the screenshot below, choose the `Ubuntu 24.04.2 LTS` version <br />
  ![image](https://github.com/user-attachments/assets/71bf3c91-0c52-4f5f-8717-ae593d59db68) <br />

- In VirtualBox, add a new VM, name it and select the correct ISO image <br />
  ![image](https://github.com/user-attachments/assets/0c0ec410-a132-4091-8459-278e8e71ff40) <br />

- A base memory of 2048MB and 1 Processor is chosen <br />
  ![image](https://github.com/user-attachments/assets/c706a1d5-ade7-4d63-ac57-a27252abc3fb) <br />

- Choose a storage size of 10GB. Only necessary files needed for this homelab project will be donwloaded and installed <br />
  ![image](https://github.com/user-attachments/assets/eaeeca03-fdc3-45fb-8ac6-ba0358988c7e) <br />

- Finish the setup process <br />
  ![image](https://github.com/user-attachments/assets/a2aabd0b-3dc7-4fb9-b410-d59f42f36bd0) <br />

- Before continuing with system updates and tool downloads, change the network adapter from NAT (default) to Brigded Adapter <br />
  ![image](https://github.com/user-attachments/assets/cc1d27a2-9adf-4f31-ba28-f528a48de827) <br />

- Once the template VM is powered on, follow the guided installation for first time setup

- 



## iSCSI Target Setup on Ubuntu Server VM




## iSCSI Initiator Setup on Lubuntu Client VM





