---
title: Securing a Linux server
author: Dan
date: 2023-03-24 23:00:00 -400
categories: [Linux, Documentation]
tags: [homelab,documentation,linux]
---


# Securing a Linux server
After creating a new Linux server, in my example it's debian/Ubuntu, log into the server with your ssh client of choice

```bash
ssh root@server-ip-address
```

# Install all updates

1: If logged in as root, don't use the sudo command 
```bash
sudo apt update && sudo apt upgrade -y
```
2: If using root, you need to create a new user
```bash
adduser "username" # change "username" 
```
Follow the prompts

3: add the new user to the sudo group
```bash
usermod -aG sudo "username"
```
Changing "username" to the one created.

Log out of thee ssh session
```bash
logout
```

Try logging in with the user just created.

```bash
ssh user@server-ip-address
```

Then try using the sudo command
```bash
sudo apt update
```

If after entering the password, it works, the user is part of the sudo group.

# Adding an ssh preshared key

Log out of the server
```bash
logout
```

On your host machine create a ssh key pair

```bash
ssh-keygen -t ed25519 -C "comment for file"
```

Change, comment on file to whatever you want to show so it's identifiable to you.

Follow the prompts, use a passphrase if you want, name the file or use the default.

Upload the public key to your server. There's multiple ways to do this, I will add in the others at a later time.

```bash
ssh-copy-id username@server-ip-address
```

Or if you named the public key to a different name 

```bash
ssh-copy-id -i ~/.ssh/name-of-file.pub
```

The try logging into the server again
If it logs straight in without asking for a password or if it asks for the passphrase you used making the ssh key, your all good.

Now edit the sshd_config to disable root and password login.

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config
```
Change the following lines
change 
PasswordAuthentication yes 
to PasswordAuthentication no

Change
allowrootlogin yes
To allowrootlogin no
Change the port if desired

```bash
sudo systemctl restart sshd
```
Open a new ssh session without closing the one you are logged into the server with and try logging in.

If you are able to log in, Great 👍

# Set up a firewall

```bash
sudo apt update && sudo apt install ufw
```
After it is installed check the status

```bash
sudo ufw status
```
At this point it should say it's disabled.
Before enabling it open the ports needed to access it. ssh at minimum.

```bash
sudo ufw allow ssh # or port
sudo ufw allow 22
```
Enable the firewall
```bash
sudo ufw enable
```

Don't log out of this ssh session, start a new one to be safe
If you are able to log in Great 👍 if not you will need to make sure that you have the right port open with the original session.

# Enable automatic updates

In a no production environment, this should be fine, an update can potentially break something, so if the system is mission critical, enable at your own risk assessment.
Updates are essential, but if it's mission critical system, install them on a test server first.
(My opinion anyway)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install unattended-updates
dpkg-reconfigure --priority low unattended-upgrades
```

That's all for now, I will update this document with any additional details or updates.

