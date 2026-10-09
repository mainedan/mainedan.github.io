---
title: Helpful Windows Commands
author: Mainedan
date: 2023-03-27 23:00:00 -400
categories: [Windows, Documentation]
tags: [homelab,documentation,windows]
---

# Useful Windows Commands for PowerShell and CMD

This guide covers essential commands for managing disks using **DiskPart**, a powerful command-line utility in Windows.

## Using DiskPart for Disk Management

**DiskPart** is used for partitioning, formatting, and cleaning disks.

1. Open a command prompt and type:
   `diskpart`

2. Use the following commands within the DiskPart utility:

- `list disk`: Lists all physical and logical disks connected to your system.
- `select disk`: Selects a specific disk to perform operations on (use the number provided by the `list disk` command).
- `clean`: Removes all partition and volume information from the selected disk, effectively wiping the partition table.
- `create partition primary`: Creates a new primary partition on the selected disk.
- `format FS=NTFS`: Formats the selected partition using the **NTFS** file system.
- `format --help`: Displays additional options and use cases for the format command.
