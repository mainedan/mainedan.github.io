---
title: Proxmox CloudInit virtual machine template
author: Dan
date: 2023-12-22 06:00:00 -400
categories: [Proxmox, Documentation]
tags: [homelab,documentation,proxmox,linux]
---

# Create a Proxmox CloudInit Virtual Machine Template

This guide explains how to create a CloudInit-enabled virtual machine template in Proxmox. Using templates allows you to quickly clone standardized VMs with pre-configured settings like SSH keys and usernames.

## Step 1: Create the Base VM

Follow these settings in the Proxmox Virtual Machine wizard:

- **General Tab**:
    - **VM ID**: Assign a unique ID (e.g., `5000`).
    - **Name**: Give it a descriptive name (e.g., `example-linux-base-template`).
    - Keep all other settings at default.

- **OS Tab**:
    - Select **Do not use any media**.
    - **Guest OS**: Ensure this is set to **Linux**.

- **System Tab**:
    - Leave all settings at default.
    - You may enable the **QEMU Guest Agent**.

- **Disks Tab**:
    - **Delete** the default disk (we will import our own later).

- **CPU Tab**:
    - Enter the desired number of CPU cores (you can adjust this later).

- **Memory Tab**:
    - Enter the desired amount of RAM (you can adjust this later).

- **Network Tab**:
    - Leave at default settings.

- **Confirm Tab**:
    - Review all settings. **Do not** select "Start after created".

## Step 2: Configure CloudInit

Once the VM is created, configure the CloudInit settings:

1. Select the VM you just created.
2. Go to the **Hardware** tab.
3. Select **Add** > **CloudInit Drive**.
4. Select the storage where you want this drive to reside and click **OK**.
5. Navigate to the **CloudInit** tab.
6. Enter the required information:
    - **User**: Define the default username.
    - **Password**: Set a default password (optional).
    - **SSH Keys**: Add your public SSH key (highly recommended).
    - **IP Config**: Select **DHCP** (or your preferred network configuration).
7. Right-click the VM and select **Convert to Template**.

## Step 3: Cloning the Template

When you need a new VM from this template:

1. Right-click the selected template and select **Clone**.
2. Enter the desired name and info for the new VM.
3. **Mode**: Choose **Full Clone** (unless you specifically require a Linked Clone).
4. Select the target storage and format.

## Step 4: Import the Cloud Image

To use a standard cloud image (like Debian), you must import the `.qcow2` file to your storage.

1. Find the desired cloud image. For example, the latest Debian bookworm image:
   [https://cloud.debian.org/images/cloud/bookworm/latest/](https://cloud.debian.org/images/cloud/bookworm/latest/)
2. Right-click the `.qcow2` file and select **Copy link**.
3. Select the PVE node where the VM is located and open the **Shell**.
4. Use `wget` to download the image:
   ```bash
   wget https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2
   ```
5. Import the disk to the VM (replace `9001` with your VM ID and `local` with your storage name):
   ```bash
   qm importdisk 9001 debian-12-generic-amd64.qcow2 local --format qcow2
   ```
6. Once the import is complete, the disk will appear as an **Unused Disk 0** in the VM hardware.
7. Select the VM, go to the **Hardware** tab, select **Unused Disk 0**, click **Edit**, and set the appropriate settings (e.g., SCSI or VirtIO Block).
8. Click **Add**.

## Step 5: Finalize and Verify

1. Start the VM.
2. Select the **Console** tab and log in.
3. Run `ip a` to verify the IP addresses assigned by DHCP.
4. Try to log in via **SSH** from your workstation to ensure your keys are working correctly.
