---
title: Helpful Windows Commands
author: Dan
date: 2023-03-27 23:00:00 -400
categories: [Windows, Documentation]
tags: [homelab,documentation,windows]
---


# Useful windows commands for PowerShell/CMD prompt
DiskPart

```cmd
diskpart
1- list disk #lists system disks
2- select disk #select the disk you want to work on by selecting the number shown with list disk command 
3- clean #removes partition table
4- create partition primary #creates a primary partition
5- format FS=NTFS #formats the drive using NTFS
6- format --help #shows additional format options and use cases
```
