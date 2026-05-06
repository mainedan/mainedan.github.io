---
title: Jellyfin Proxmox LXC with SMB share
author: Dan
date: 2026-05-06 04:00:00 -400
categories: [Proxmox, Jellyfin, Documentation]
tags: [homelab,documentation]
---

# Jellyfin setup

```bash
pct create 101 /var/lib/vz/template/cache/ubuntu-20.04-standard_20.04-1_amd64.tar.gz --hostname my-container --memory 2048 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.100,gw=192.168.1.1
```

This will create an LXC container with the
    * VMID: 101
    * Template: Ubuntu 20.04
    * Hostname: Jellyfin
    * Memory: 2048 MB
    * Network: Connected to bridge vmbr0 with a static IP of 192.168.1.100 and gateway 192.168.1.1

Adjust as nessesary, or use the Proxmox GUI

Start the container

```bash
pct start 101
```

This will start the proxmox VM 101

Log into the jellyfin server and update and install curl and cifs-utils

```bash
apt update && apt upgrade -y && apt install curl cifs-utils -y
```
Add SMB credentials

```bash
nano .smbcredentials
```

Add this to the file

```bash
username=SMB_Username
password=SMB_Password
```

Mount the SMB share in Proxmox in /etc/fstab ,  add to the bottom of the file

```bash
nano /etc/fstab


//SMB_Address/SMB_Directory /mnt/media cifs _netdev,x-systemd.automount,noatime,uid=100000,gid=110000,dir_mode=>
```



Install Jellyfin in the VM

```bash
curl -s https://repo.jellyfin.org/install-debuntu.sh -O && \
curl -s https://repo.jellyfin.org/install-debuntu.sh.sha256sum -O && \
sha256sum -c install-debuntu.sh.sha256sum
```
install-debuntu.sh: OK means the checksum is correct.

You can optionally inspect the script to see what it does before executing it:

```bash
less install-debuntu.sh
```

Then execute it with:

```bash
sudo bash install-debuntu.sh
```

Add the group lxc_shares

```bash
groupadd -g 10000 lxc_shares
```

Add the Jellyfin user to the lxc_shares group

```bash
usermod -aG lxc_shares jellyfin
```

Check your Jellyfin ID

```bash
id jellyfin
```

Reboot the server

```bash
reboot
```

Add the library to the Jellyfin server



Restart the VM

```bash
reboot
```




![Rambo-Tux](/assets/images/rambo-tux.png)
