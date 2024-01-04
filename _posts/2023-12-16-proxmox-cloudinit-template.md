---
title: Proxmox CloudInit virtual machine template
author: Dan
date: 2023-12-22 06:00:00 -400
categories: [Proxmox, linux, Documentation]
tags: [homelab,documentation,proxmox,linux]
---

# Create a basic template

Creat VM

* On Create: Virtual Machine wizard - General Tab
    > Enter desired VM ID: exampaple - 5000

    > Enter a name: exaple linux-base-template

    > Everything else at defaults for this tab


---

* On the OS Tab
    > Select Do not use any media

    > Guest OS: make sure it is set to Linux

---

* On the System Tab
    > Leave enverything at the defaults

    > Can set the QEMU Agent on

---

* On the Disks tab
    > Delete the disk

---

* On the CPU tab
    > Enter the desired CPU cores, can adjust to this later if needed

---

* On the Memory tab
    > Enter the desired memory, can adjust later if needed

---

* On the Network tab
    > leave at defaults

---

* On the Confirm tab
    > Make sure everything looks ok, go back and adjust as needed, do not select Start after created

---

* Select the VM you created
    > select the hardware Tab
    
    >Select Add - CloudInit Drive
    
    > Select the storage that you want this on, then OK
    
    > Select CloudInit tab and enter the info you need there (an SSH key is very helpfull) add username and password. Also helpfull to select DHCP on the IP Config area
    
    > Right clock on the VM and select Convert to Template
    
    > Right click on the selected template and select clone
    
    > Enter the desired info
    
    > Mode: use Full Clone (can use linked clone if desired)
    
    > Select the target storage and format

---

* Go to the desired cloud image you want for the base template

    > debian can be found here
    
    > https://cloud.debian.org/images/cloud/
    
    > go to the latest version (bookworm is now)
    
    > go to the latest page and find the one that verion of qcow2 that is needed for your system, amd64 is what I am using.
    
    > this is the current one now
    
    > https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2
    
    > Right click on the file and select copy link

---

* Select the PVE node the VM is on and open the shell

    > at the command prompt enter wget https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2
    >
    > it will dowload the file to the system.
    >
    > then enter 
    >
    ```bash 
    qm importdisk 9001 debian-12-generic-amd64.qcow2 local --format qcow2
    ```
    >
    >
    > adjust to your system setup
    >
    > It should transfer the file to the VM and at the end show; example - Successfully imported disk as 'unused0:local:104/vm-104-disk-0.qcow2'

---

* Select the VM

    > Select the hardware tab
    >
    > select the Unused Disk 0 then select Edit
    >
    > change the setting to what you need
    >
    > Select Add

---

* Start the VM

    > Select the Console tab and login
    >
    > enter ip a to get the IP addresses the VM is set to use
    >
    > Try to log in with SSH to make sure yor key is working

