---
title: Proxmox Windows VM install
author: Dan
date: 2023-03-31 10:00:00 -400
categories: [Proxmox, Windows, Documentation]
tags: [homelab,documentation,proxmox,windows]
---

# Proxmox Windows VM install

## Download Pertinent ISOs

Download the pertinent ISOs (add links here).

## Create a VM

1. **VM ID**: Choose a unique ID.
2. **Name**: Enter a name.
3. **ISO Selection**: Select the ISO Windows image.
4. **OS Selection**: Select Microsoft Windows type and date.
5. **System**: Select **qemu agent**, **TPM**, and **BIOS** as needed, and use **VirtIO SCSI**.
6. **Location**: Put the TPM and UEFI location to where the VM will be installed.
7. **Machine Type**: Set machine type to **q35**.
8. **Disks**: Select **discard** if using an SSD, and select **write back** under Cache.
9. **Bus Device**: Set to **virtio block**.
10. **Cores and Memory**: Select the desired cores and memory.
11. **CPU Type**: Set CPU type to **host**.

## Before Starting the VM

Before starting the VM, select the VM hardware and add a **CD-ROM drive** with the **VirtIO win ISO**.

## Install OS

Install the OS as usual. If the drivers for the disks are not found:
1. Browse to **amd64/windows** version.
2. Load the network driver if desired.

## After Windows Installation

Update the drivers for those that didn't load. You can browse to the drivers needed or use the installer on the disk.

## Update Windows and Configuration

Update Windows and set up as needed.
