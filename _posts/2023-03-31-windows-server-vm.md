---
title: Proxmox Windows Server post install
author: Dan
date: 2023-03-31 11:00:00 -400
categories: [Proxmox, Windows, Documentation]
tags: [homelab,documentation,proxmox,windowsserver]
---

# Windows Server post install

Post install

* Go to device manager

    > If any devices not loading right click/properties on the ones that are not.

    > Update Driver

    > Browse my computer for drivers

    > Browse and go to the cd for the virtio drivers

    > Make sure to include subfolders, select next

    > Continue until all of the drivers are installed

---

* Go to Update and Security

    > Check for updates and install all of them

    > Reboot when done, recheck when back in windows, do all until no more are available

---

* Make sure time and date are correct, and time zone

---

* Go to:

    > Server Manager - Local server

    > Set static IP by clicking on IPv4 address assigned by DHCP

    > Right click on the ethernet instance, select status

    > Click properties, select IPv4 protocol then properties

    > Fill out IP information, click ok and close the network connections panel

    > Enable remote desktop

    > Change Computer name to one that will work

---

* Restart server to active the changes


