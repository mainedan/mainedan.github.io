---
title: Automating a Homelab and VPS with Ansible, Proxmox, and Tailscale
author: Mainedan
date: 2026-11-22 03:46:00 -0500
categories: [Homelab, Automation]
tags: [ansible, proxmox, tailscale, linux, documentation]
---

Managing a mixed homelab of **Proxmox VE** nodes, **Debian 13** servers, **Windows Server 2022** domain controllers, and an external **VPS** quickly turns into repetitive manual maintenance. This guide documents a clean, Git-backed **Ansible** repository structure that automates OS patching, SSH key synchronization, Docker Compose updates, and **Proxmox LXC/VM provisioning** with automatic **Tailscale** onboarding.

## Step 1: Repository Structure and Control Node Setup

On a **Fedora** control node, install **ansible-core**, **python3-winrm** (for Windows targets), and the required **Ansible Galaxy** collections:

```bash
sudo dnf install ansible-core python3-winrm
ansible-galaxy collection install -r requirements.yml
```

Organize the repository with OS-specific **group_vars**, a shared `files/authorized_keys` file, and modular playbooks:

```text
.
├── ansible.cfg
├── requirements.yml
├── files/
│   └── authorized_keys
├── inventory/
│   ├── hosts.yml
│   └── group_vars/
│       ├── all/
│       │   └── vault.yml       # Encrypted with Ansible Vault
│       ├── linux.yml
│       ├── proxmox.yml
│       ├── vps.yml
│       └── windows.yml
└── playbooks/
    ├── bootstrap-debian.yml
    ├── build-vm-template.yml
    ├── create-lxc.yml
    ├── create-vm.yml
    ├── destroy-proxmox-guest.yml
    ├── ping-all.yml
    ├── push-ssh-keys.yml
    ├── update-debian.yml
    ├── update-docker-stacks.yml
    ├── update-proxmox.yml
    └── update-windows.yml
```

## Step 2: Secure Secrets with Ansible Vault

All privileged passwords and API keys live in an AES-256 encrypted **vault.yml** file (`inventory/group_vars/all/vault.yml`), while the decryption password stays strictly local in `.vault_pass` (excluded via `.gitignore`).

```bash
# Create the local vault password file (never commit this)
echo '<SECRET>' > .vault_pass
chmod 600 .vault_pass

# Edit encrypted secrets
ansible-vault edit inventory/group_vars/all/vault.yml
```

Store separate credentials for each environment along with an optional reusable **Tailscale Auth Key** (non-ephemeral, 90-day expiry):

```yaml
vault_lan_become_password: "<SECRET>"
vault_vps_become_password: "<SECRET>"
vault_windows_password: "<SECRET>"
vault_tailscale_authkey: "<SECRET>"
```

> Always verify that `vault.yml` starts with `$ANSIBLE_VAULT;1.1;AES256` and that `.vault_pass` is matched by `.gitignore` (`git check-ignore -v .vault_pass`) before pushing to GitHub.
{: .prompt-warning }

## Step 3: Build a Debian 13 Cloud-Init VM Template on Proxmox

Before cloning VMs, run `playbooks/build-vm-template.yml` to enable **snippets** on `local` storage, download the official **Debian 13 genericcloud** image, and build template `9000` with **QEMU Guest Agent** enabled:

```bash
# Build template 9000 on Proxmox nodes (skips if already present)
ansible-playbook playbooks/build-vm-template.yml

# Force a fresh rebuild with the latest Debian 13 cloud image
ansible-playbook playbooks/build-vm-template.yml -e rebuild_template=true
```

## Step 4: Provision Proxmox LXCs and VMs with Tailscale

The provisioning playbooks enforce a clean ID split: **LXC containers** use IDs `100–199` (auto-detected via `pvesh get /cluster/nextid` when the ID prompt is left blank), while **QEMU VMs** use IDs `200+`.

* **LXC Provisioning (`playbooks/create-lxc.yml`):** Creates an unprivileged Debian 13 container on `<HOMELAB-IP>/24` (`<DOMAIN>`), injects `/dev/net/tun` passthrough into `/etc/pve/lxc/<ct_id>.conf` before boot, installs `sudo`, `curl`, `openssh-server`, and **Tailscale**, pushes SSH keys, and auto-joins the tailnet using `vault_tailscale_authkey`.
* **VM Provisioning (`playbooks/create-vm.yml`):** Clones template `9000`, resizes the boot disk, attaches a custom **cloud-init** snippet installing `qemu-guest-agent`, `curl`, `sudo`, and **Tailscale**, and authenticates to the tailnet on first boot.
* **Safe Teardown (`playbooks/destroy-proxmox-guest.yml`):** Auto-detects whether an ID is an LXC or VM, blocks deletion of protected production IDs (such as domain controllers, backup servers, and template `9000`), and requires typing `DESTROY` before purging the guest and any leftover cloud-init snippet.

```bash
# Provision a new LXC (press Enter on ID prompt for next available 100-199)
ansible-playbook playbooks/create-lxc.yml

# Provision a new VM (ID 200+)
ansible-playbook playbooks/create-vm.yml

# Safely destroy a test LXC or VM
ansible-playbook playbooks/destroy-proxmox-guest.yml
```

> Keep **Ephemeral** turned **OFF** when generating your reusable Tailscale auth key so persistent homelab LXCs and VMs are not automatically removed from your tailnet during reboots or maintenance.
{: .prompt-tip }

## Step 5: Everyday Maintenance and Verification

Use these playbooks for routine connectivity checks, staged OS updates, and modular **Docker Compose** stack refreshes across LAN servers and `<VPS-HOST>`:

```bash
# Test SSH (Linux/Proxmox) and WinRM (Windows) connectivity everywhere
ansible-playbook playbooks/ping-all.yml

# Patch Debian LAN hosts and <VPS-HOST> (add -e auto_reboot=true to reboot)
ansible-playbook playbooks/update-debian.yml

# Patch Proxmox VE nodes one at a time (never auto-reboots)
ansible-playbook playbooks/update-proxmox.yml

# Patch Windows Server domain controllers one at a time
ansible-playbook playbooks/update-windows.yml

# Pull and recreate modular Docker Compose stacks
ansible-playbook playbooks/update-docker-stacks.yml
ansible-playbook playbooks/update-docker-stacks.yml -e target_host=<VPS-HOST>
```

> Validate all playbooks offline before committing changes with `ansible-playbook --syntax-check playbooks/*.yml`.
{: .prompt-info }
