---
title: Proxmox Windows VM install
author: Dan
date: 2023-03-31 10:00:00 -400
categories: [Proxmox, Documentation]
tags: [homelab,documentation,proxmox]
---

In proxmox
	• Download pertinent ISOs (add links here)
	• Create a VM
		○ VM ID
		○ Name - next
		○ Select the ISO windows image
			§ Select Microsoft Windows type and date
		○ Under system, select qemu agent, TPM and BIOS is needed and VirtIO SCSI
			§ Put the tpm and uefi location to where the vm is stored
			§ Set machine type to q35
		○ Under disks, select discard if using an SSD, select write back under Cache
			§ Bus device set to virtio block
		○ Select the cores and memory wanted
		○ Set CPU type to host
		○ Set the memory
		○ Finish
	• Select hardware under the VM and add a CDRom drive and select the VirtIO win ISO
	• Install OS as usual, if it does not find the drivers for the disks
		○ Browse - amd64/windows version
		○ Load network driver if desired
	• After it has installed
		○ Update the drivers for the ones that didn't load
Can browse to the drivers needed or use the installer on the disk