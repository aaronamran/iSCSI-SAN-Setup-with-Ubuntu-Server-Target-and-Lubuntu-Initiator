# iSCSI-SAN-Setup-with-Ubuntu-Server-Target-and-Lubuntu-Initiator

This write-up documents a practical SAN-like storage setup project using VirtualBox to simulate a Storage Area Network (SAN) environment with free, open-source tools. A Ubuntu Server VM is configured as the iSCSI target using `targetcli`, Logical Volume Manager (LVM), and additional virtual storage disks. A Lubuntu VM acts as the iSCSI initiator, connecting to the target to access and mount remote block storage over the network. This project demonstrates core concepts of enterprise SAN architecture including target and LUN creation, iSCSI discovery and login, persistent mounting via `/etc/fstab`, and automation using systemd, all within a lightweight lab suitable for a single-machine setup. The goal is to gain hands-on experience with iSCSI storage provisioning and emulate real-world infrastructure commonly found in datacenter environments. 

![iSCSI_SAN_Ubuntu_Server_target_Lubuntu_initiator](https://github.com/user-attachments/assets/93983326-5947-4e82-b75d-fb0616800133) <br />

1. [Configuration of VMs in VirtualBox](#configuration-of-vms-in-virtualbox)
2. [iSCSI Target Setup on Ubuntu Server VM](#iscsi-target-setup-on-ubuntu-server-vm)
3. [iSCSI Initiator Setup on Lubuntu Client VM](#iscsi-initiator-setup-on-lubuntu-client-vm)
4. [Automation and Testing of iSCSI](#automation-and-testing-of-iscsi)


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

- Add a second virtual disk of 10 GB size to be used as the Logical Unit Number (LUN). In VirtualBox, go to the Settings of Ubuntu Server and navigate to Storage. Click the Add Hard Disk icon <br />
  ![image](https://github.com/user-attachments/assets/7379978f-b5cf-4bfc-bd7d-e6631ecc7634) <br />

- In the pop-up, choose Create Disk Image File. Then keep as VDI (VirtualBox Disk Image) and set the size to 10 GB <br />
  ![image](https://github.com/user-attachments/assets/ede151a0-892f-400e-8034-d65868de5bbd) <br />
  
- Once the template VM is powered on, follow the guided installation for first time setup. For the sake of this project, a ubuntu server account name and password of sysadmin will be used



## iSCSI Target Setup on Ubuntu Server VM

- Login to the Ubuntu Server VM and install required packages using the following commands
  ```
  sudo apt update
  sudo apt install targetcli-fb lvm2
  ```
  ![image](https://github.com/user-attachments/assets/537ba567-20b4-441e-9ad6-bd13f4beef1f) <br />

- Then prepare a volume with LVM
  ```
  # Identify your extra disk (e.g., /dev/sdb)
  lsblk
  
  # Create LVM Physical Volume
  sudo pvcreate /dev/sdb
  
  # Create a Volume Group
  sudo vgcreate vg_iscsi /dev/sdb
  
  # Create a Logical Volume
  sudo lvcreate -L 4G -n lv_storage vg_iscsi
  ```
  ![image](https://github.com/user-attachments/assets/f60245c8-1afb-4c3b-bf5c-e05b844e742a)


- Create iSCSI target with `targetcli` interactive shell. This specific step requires the IQN (iSCSI Qualified Name) of the Lubuntu Client iSCSI initiator. Setup the iSCSI on Lubuntu Client VM at [iSCSI Initiator Setup on Lubuntu Client VM](#iscsi-initiator-setup-on-lubuntu-client-vm) then return to this step
  ```
  sudo targetcli
  
  # Create a backstore for the LVM volume
  /backstores/block create name=block1 dev=/dev/vg_iscsi/lv_storage
  
  # Create iSCSI target
  /iscsi create iqn.2025-05.com.example:storage.target1
  
  # Create LUN
  /iscsi/iqn.2025-05.com.example:storage.target1/tpg1/luns create /backstores/block/block1
  
  # Allow initiator access (replace IP or subnet)
  /iscsi/iqn.2025-05.com.example:storage.target1/tpg1/acls create iqn.2004-10.com.ubuntu:01:327b79a5da9f
  /iscsi/iqn.2025-05.com.example:storage.target1/tpg1/portals create 0.0.0.0 3260
  
  # Save and exit
  exit
  ```
  ![image](https://github.com/user-attachments/assets/a433c331-a504-4f45-a99d-d90f268e117d) <br />

  In the provided commands, the `com.example` is a user-defined name for the iSCSI target. It is not auto-generated like the initiator IQN and it is free to be customised. IQNs follow this format: `iqn.<year>-<month>.<reversed domain>:<custom-label>` <br />

- Enable and start the target service
  ```
  sudo systemctl enable target.service
  sudo systemctl start target.service
  ```
  ![image](https://github.com/user-attachments/assets/e8be721d-f361-4f45-b1e6-7dda20c24f3b)



## iSCSI Initiator Setup on Lubuntu Client VM

- In Lubuntu Client VM, install the iSCSI tools needed
  ```
  sudo apt update
  sudo apt install open-iscsi
  ```
  ![image](https://github.com/user-attachments/assets/46e8993c-7bf2-4bf5-b92e-b864fe7aee1e) <br />

- After the installation is complete, get the IQN. Note that it requires sudo to read the file. Once obtainig the IQN, return to the `targetcli` step in [iSCSI Target Setup on Ubuntu Server VM](#iscsi-target-setup-on-ubuntu-server-vm) <br />
  ![image](https://github.com/user-attachments/assets/0682147f-da67-4aa2-befc-07a2ba3f9fd8) <br />


- Discover the target. Replace the IP address in the following command with iscsi-target (Ubuntu Server VM) IP address
  ```
  sudo iscsiadm -m discovery -t st -p 192.168.x.x
  ```
  ![image](https://github.com/user-attachments/assets/5c37d855-6930-4c10-82e7-4b6ae8063d78)

- Log in to the target using the discovered IQN
  ```
  sudo iscsiadm -m node --login
  ```
  ![image](https://github.com/user-attachments/assets/5f664b71-6d75-4f63-8378-0be73c9846e1)

- Confirm the connection and the partition
  ```
  lsblk
  
  # You should see a new block device (e.g., /dev/sdb)
  # Optionally format and mount it:
  sudo mkfs.ext4 /dev/sdb
  sudo mkdir /mnt/iscsi
  sudo mount /dev/sdb /mnt/iscsi
  df -h
  ```
  ![image](https://github.com/user-attachments/assets/bb1e3ec2-0fda-49c0-b706-b120ee429aa4)


## Automation and Testing of iSCSI

- For this section, a systemd unit or shell script that runs at boot to ensure LVM and iSCSI coming up properly will be created
- In Ubuntu Server VM, `targetcli` saves config to `/etc/target/saveconfig.json` automatically and this config is auto-loaded by the `target` service on boot
- On Lubuntu Client VM (the iSCSI initiator), use the commands to automate initiator login
  ```
  sudo iscsiadm -m node -T iqn.2025-05.com.example:storage.target1 --op update -n node.startup -v automatic
  sudo systemctl enable open-iscsi
  ```
  ![image](https://github.com/user-attachments/assets/2c90648c-bf67-41e1-9973-cefada928a89)


- To auto-mount the iSCSI disk on boot ensure the iSCSI disk is working now. Run the following command
  ```
  lsblk
  ```

- Create a permanent mount point using
  ```
  sudo mkdir -p /mnt/iscsi
  ```

- Get the UUID of the iSCSI disk as this is safer than using `/dev/sdb` which can change
  ```
  sudo blkid /dev/sdb
  ```
  ![image](https://github.com/user-attachments/assets/eee13094-a444-4f01-8289-af402d29e815)


- Open and edit `/etc/fstab` in a text editor
  ```
  sudo nano /etc/fstab
  ```

- Then add this line at the bottom using the UUID obtained previously
  ```
  UUID=315a8e52-f7cb-4b42-bf40-7c605dbe31ce  /mnt/iscsi  ext4  _netdev  0  0
  ```
  `_netdev` is used to tell the system to wait for networking before trying to mount <br />
  ![image](https://github.com/user-attachments/assets/69a8a1bd-8841-4301-a6cd-e79c076a01ba) <br />

- Before rebooting, test the fstab entry safely with
  ```
  sudo mount -a
  ```

- Then verify with
  ```
  df -h
  ```
  ![image](https://github.com/user-attachments/assets/ab65d091-e150-4f92-80d1-11cb91dbc15d)


- Reboot both VMs using
  ```
  sudo shutdown -r
  ```

- Ensure that the initiator automatically connects and mounts the iSCSI LUN by running
  ```
  sudo iscsiadm -m session
  ```
  ![image](https://github.com/user-attachments/assets/67615307-3c9f-4026-9904-15cc28f344d5)

- Trying to delete the session while it is still running is not doable. The following command should be used in order
  ```
  sudo iscsiadm -m node --logout
  sudo iscsiadm -m node -o delete
  ```
  ![image](https://github.com/user-attachments/assets/1b57cf5d-4f7d-4221-9fda-f0c304930cd0) 


- To check for errors in logs use
  ```
  journalctl -b | grep iscsi
  ```
  An example of iSCSI target session being unavailable due to change of IP address is shown in the following screenshot <br />
  ![image](https://github.com/user-attachments/assets/da00a7dd-ec9a-40d2-8826-ce6a154751d2) <br />

  To troubleshoot this case, delete the old iSCSI target session
  ```
  sudo iscsiadm -m node -o delete
  ```
  
  Then discover the new target IP
  ```
  sudo iscsiadm -m discovery -t sendtargets -p 192.168.1.5
  ```

  Login to the target again
  ```
  sudo iscsiadm -m node --login
  ```

  Make this login persistent again. Replace the IQN if it differs
  ```
  sudo iscsiadm -m node -T iqn.2025-05.com.example:storage.target1 -p 192.168.1.5 --op update -n node.startup -v automatic
  ```
  ![image](https://github.com/user-attachments/assets/fb219b10-b0e1-457f-9138-d4f4cb466200)


  
  












