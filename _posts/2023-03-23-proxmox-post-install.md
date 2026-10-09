---
title: Proxmox Post-Install
author: Dan
date: 2023-03-23 10:00:00 -400
categories: [Proxmox, Documentation]
tags: [homelab,documentation,proxmox]
---

# Proxmox Post Install

## Log into Proxmox

Log into Proxmox using your SSH client:

```bash
ssh root@(ip address)
```

## Set Repositories

Open the sources list file:

```bash
nano /etc/apt/sources.list
```

Make sure these repositories are listed:

```plaintext
deb http://ftp.debian.org/debian bullseye main contrib 
deb http://ftp.debian.org/debian bullseye-updates main contrib 
```

**Note**: The following repository is **NOT recommended for production use**:

```plaintext
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription
```

## Security Updates

Add the following line to your sources list:

```plaintext
deb http://security.debian.org/debian-security bullseye-security main contrib
```

## Comment out the Subscription

Remove the enterprise repository file:

```bash
rm /etc/apt/sources.list.d/pve-enterprise.list
```

## Update the System

Run the following commands to update your system:

```bash
apt update && apt upgrade
```

## Update the Container Library

Update the container images:

```bash
pveam update
```

## View Available Images

To view the list of available images, run:

```bash
pveam available
```

## Remove License Banner

To remove the "No valid subscription" banner, follow these steps:

```bash
ssh root@(ip address)
cd /usr/share/javascript/proxmox-widget-toolkit
cp proxmoxlib.js proxmoxlib.js.bak
nano interfaces
```

In `nano`, press **ctrl+w** and search for **No valid subscription**.

Change the following line to:

```plaintext
void({ //Ext.Msg.show({ 

  title: gettext('No valid subscription'), 
```

Restart the `pveproxy` service:

```bash
systemctl restart pveproxy.service
```

## Remove and Resize Local Drive

Commands for single drive storage:

```bash
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```
