---
title: Proxmox Post-Install
author: Dan
date: 2023-03-23 10:00:00 -400
categories: [Proxmox, Documentation]
tags: [homelab,documentation,proxmox]
---

# Proxmox Post Install

Log into proxmox 

----------------- 

Set repositories 

```bash
nano  /etc/apt/sources.list 
```

Make sure these repositories are listed
 
```plaintext
deb http://ftp.debian.org/debian bullseye main contrib 

deb http://ftp.debian.org/debian bullseye-updates main contrib 

PVE pve-no-subscription repository provided by proxmox.com 
```
NOT recommended for production use 

```plaintext
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription 
```

-----------------  

## Security Updates 

Add this line

```plaintext
deb http://security.debian.org/debian-security bullseye-security main contrib 
```
----------------- 

## Comment out the subscription  

 
```plaintext
/etc/apt/sources.list.d/pve-enterprise.list 
```
 
## Then update the system 
 
```bash
apt update && apt upgrade 
```

## Update the container library 

```bash
pveam update 
```

## To view the list of available images run 

```bash
pveam available 
```
 

## Commands for Remove License Banner: 

----------------- 

```bash
ssh root@(ip address) 

cd /usr/share/javascript/proxmox-widget-toolkit 

cp proxmoxlib.js proxmoxlib.js.bak  

nano interfaces 
```

In nano hit

ctrl+w and search No valid subscription

change to

```plaintext
void({ //Ext.Msg.show({ 

  title: gettext('No valid subscription'), 
```
 
 Restart the pveproxy service

```bash
systemctl restart pveproxy.service
```

## Remove and Resize local drive

Commands for single drive storage: 

----------------- 

```bash
lvremove /dev/pve/data 

lvresize -l +100%FREE /dev/pve/root 

resize2fs /dev/mapper/pve-root 
```
