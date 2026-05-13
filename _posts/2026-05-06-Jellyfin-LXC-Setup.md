---
title: Jellyfin Proxmox LXC with SMB share
author: Dan
date: 2026-05-06 04:00:00 -400
categories: [Proxmox, Jellyfin, Documentation]
tags: [homelab,documentation]
---
# SMB setup and fstab on proxmox host

Add SMB credentials as root on proxmox host

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


//SMB_Address/SMB_Directory /host-mount-dir/host-mount-sub-dir cifs credentials=/root/.smbcredentials,_netdev,x-systemd.automount,noatime,uid=100000,gid=110000,dir_mode=0755,file_mode=0644
```

Adjust the uid and gid to match the uid and group of the user you want to have access to the share for example;

```bash
//SMB_Address/SMB_Directory /host-mount-dir/host-mount-sub-dir cifs credentials=/root/.smbcredentials,uid=100000,gid=101001 0 0
```

This will mount the files with the uid=1001 and gid=1001 from the VM, to match the root user uid and gid it would be 100000

Make the directories on the host to match the fstab for example;

```bash
mkdir -p /mnt/media/video/movie

mkdir -p /mnt/media/video/tv_shows

mkdir -p parent_dir/{child1,child2,child3} 

```

Reload the deamon and mount the share

```bash
systemctl daemon-reload

mount -a
```

# LXC Setup

```bash
pct create 101 /var/lib/vz/template/cache/ubuntu-20.04-standard_20.04-1_amd64.tar.gz --hostname jellyfin --memory 2048 --net0 name=eth0,bridge=vmbr0,ip=192.168.1.100,gw=192.168.1.1
```

This will create an LXC container with the
    * VMID: 101
    * Template: Ubuntu 20.04
    * Hostname: Jellyfin
    * Memory: 2048 MB
    * Network: Connected to bridge vmbr0 with a static IP of 192.168.1.100 and gateway 192.168.1.1

Adjust as necessary, or use the Proxmox GUI

Add to the container config

```bash
nano /etc/pve/lxc/101.conf

mp0: /mnt/pve/smb-location,mp=/new-directory-for-media-files
```

Start the container

```bash
pct start 101
```

This will start the proxmox VM 101

Log into the jellyfin server and update and install curl and cifs-utils

```bash
apt update && apt upgrade -y && apt install curl cifs-utils -y
```
Check the share for files and verify the permisions

```bash
ls -al /new-directory-for-media-files
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

Add the group lxc_shares (optional)

```bash
groupadd -g 10000 lxc_shares
```

Add the Jellyfin user to the lxc_shares group (optional)

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






![Rambo-Tux](/assets/images/rambo-tux.png)
