---
title: Securing a Linux server
author: Dan
date: 2023-03-24 23:00:00 -400
categories: [Linux, Documentation]
tags: [homelab,documentation,linux]
---


# Securing a Linux server
After creating a new Linux server, in my example it's debian/Ubuntu, first install all updates

1: If logged in ar root, don't use the sudo command 
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
