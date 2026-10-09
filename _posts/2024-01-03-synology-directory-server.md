---
title: Synology AD:DC Setup
author: Mainedan
date: 2024-01-03 19:00:00 -400
categories: [Synology, Active_Directory, Documentation]
tags: [homelab,documentation,synology,active_directory]
---

# Use synology NAS as an AD:DC

## Install Synology Directory Server

Go to the **Package Manager** and install **Synology Directory Server**.

## Set a Static IP

Set a **static IP** in the **Networking** control panel tab:

1. Select **Control Panel** > **Network** > **Network Interface**.
2. Select the connection to edit (e.g., **LAN 1**) then select **Edit**.
3. Select **Use manual configuration** and enter the IP address for the static IP.
4. Example (change to your network settings):
    * **IP Address**: `192.168.1.100`
    * **Subnet Mask**: `255.255.255.0`
    * **Gateway**: `192.168.1.1`
    * **DNS Server**: `1.1.1.1`

## Open Synology Active Directory

Open **Synology Active Directory** and follow the setup wizard.
