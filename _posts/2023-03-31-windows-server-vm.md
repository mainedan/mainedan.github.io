---
title: Proxmox Windows Server post install
author: Mainedan
date: 2023-03-31 11:00:00 -400
categories: [Proxmox, Windows, Documentation]
tags: [homelab,documentation,proxmox,windowsserver]
---

# Windows Server post install

## Initial Driver Installation

1. Open **Device Manager**.
2. If any devices are not loading, right-click the ones that are not and select **Properties**.
3. Select **Update Driver**.
4. Choose **Browse my computer for drivers**.
5. Browse and go to the CD for the **VirtIO drivers**.
6. Make sure to include subfolders and select **Next**.
7. Continue until all of the drivers are installed.

## Updates

1. Go to **Update and Security**.
2. Check for updates and install all of them.
3. Reboot when done, then recheck when back in Windows. Repeat until no more updates are available.

## Time and Date

Make sure the **time and date** are correct, and the **time zone** is set properly.

## Network and Remote Desktop

1. Go to **Server Manager** -> **Local Server**.
2. Set a **static IP** by clicking on the **IPv4 address** assigned by DHCP.
3. Right-click on the ethernet instance and select **Status**.
4. Click **Properties**, select **IPv4 protocol**, then **Properties**.
5. Fill out the IP information, click **OK**, and close the network connections panel.
6. Enable **Remote Desktop**.
7. Change the **Computer name** to one that will work.

## Finalize

Restart the server to **activate** the changes.
