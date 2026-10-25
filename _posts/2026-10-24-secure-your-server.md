---
title: Securing a Linux server
author: Mainedan
date: 2026-10-24 23:00:00 -400
categories: [Linux, Documentation]
tags: [homelab,documentation,linux]
---


## Securing a Linux server
After creating a new Linux server (Debian/Ubuntu, Fedora/RHEL, or Arch Linux), log into the server with your **SSH** client of choice:

```bash
ssh root@server-ip-address
```

## Step 1: Install all updates and create a non-root user

1: Update all system packages (if logged in as `root`, you can omit `sudo`):

**Debian / Ubuntu:**
```bash
sudo apt update && sudo apt upgrade -y
```

**Fedora / RHEL:**
```bash
sudo dnf upgrade --refresh -y
```

**Arch Linux:**
```bash
sudo pacman -Syu
```

2: If using `root`, create a new regular user:

**Debian / Ubuntu:**
```bash
adduser username # change "username"
```

**Fedora / Arch Linux:**
```bash
useradd -m -s /bin/bash username
passwd username
```
Follow the prompts to set a strong password.

3: Add the new user to the sudoers group (`sudo` on Debian/Ubuntu, `wheel` on Fedora/Arch):

**Debian / Ubuntu:**
```bash
usermod -aG sudo username
```

**Fedora / Arch Linux:**
```bash
usermod -aG wheel username
```
*(Note for Arch Linux: Ensure `sudo` is installed with `pacman -S sudo` and uncomment `%wheel ALL=(ALL:ALL) ALL` using `EDITOR=nano visudo`.)*

Log out of the SSH session:
```bash
logout
```

Try logging in with the user just created:

```bash
ssh username@server-ip-address
```

Then verify `sudo` access:

```bash
sudo whoami
```

If after entering your password it outputs `root`, your user has `sudo` privileges.

## Step 2: Adding an SSH preshared key

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
ssh-copy-id -i ~/.ssh/name-of-file.pub username@server-ip-address
```

Then try logging into the server again.
If it logs straight in without asking for a password or if it asks for the passphrase you used making the ssh key, you're all good.

### Using an SSH Config File

To make connecting easier (especially if you use a custom SSH key name or non-standard port), you can configure an SSH config file on your local machine so you don't have to type the username, IP address, port, or key path every time.

On your **host (client) machine**, create or edit `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Add an entry for your server:

```text
Host myserver
    HostName server-ip-address
    User username
    Port 22
    IdentityFile ~/.ssh/name-of-file
    IdentitiesOnly yes
```

Set the proper permissions on the config file:

```bash
chmod 600 ~/.ssh/config
```

Now you can log into your server simply by typing:

```bash
ssh myserver
```

### Hardening `sshd_config`

Now edit `/etc/ssh/sshd_config` to disable root login, disable password authentication, and tighten SSH access.

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config
```

Update or add the following settings:

```text
# Disable password login and PAM challenge-response passwords
PasswordAuthentication no
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no

# Disable root login
PermitRootLogin no

# Limit authentication attempts per connection
MaxAuthTries 3

# Explicitly whitelist which users are allowed to SSH in
AllowUsers username

# Disable unnecessary forwarding features (unless needed)
X11Forwarding no
AllowTcpForwarding no

# Optional: Change the default SSH port
# Port 2222
```

Before restarting the SSH service, always test the configuration syntax to make sure there are no typos that could lock you out:

```bash
sudo sshd -t
```

If it returns no output, the syntax is valid. Restart the SSH daemon:

```bash
# Debian / Ubuntu:
sudo systemctl restart ssh

# Fedora / Arch Linux:
sudo systemctl restart sshd
```

Open a new SSH session **without closing the one you are currently logged into** and try logging in.

If you are able to log in, Great 👍

### Two-Factor Authentication (2FA) for SSH or Sudo

For an extra layer of security, you can require Two-Factor Authentication (2FA). There are two common ways to do this:

#### Option A: Hardware Security Keys (YubiKey / FIDO2)
OpenSSH 8.2+ natively supports FIDO2 hardware keys (`ed25519-sk`), requiring a physical tap on your security key to log in:

```bash
ssh-keygen -t ed25519-sk -C "yubikey-ssh"
ssh-copy-id -i ~/.ssh/id_ed25519_sk.pub username@server-ip-address
```

#### Option B: TOTP Authenticator App (Google Authenticator / Authy / Bitwarden)

1. Install the Google Authenticator PAM module:

**Debian / Ubuntu:**
```bash
sudo apt install libpam-google-authenticator -y
```

**Fedora / RHEL:**
```bash
sudo dnf install google-authenticator -y
```

**Arch Linux:**
```bash
sudo pacman -S libpam-google-authenticator
```

2. Run the initializer as your regular user (not root) and scan the QR code with your authenticator app:
```bash
google-authenticator
```
*(Recommended answers: `y` for time-based tokens, `y` to update `~/.google_authenticator`, `y` to disallow multiple uses, `n` for time skew unless needed, `y` to enable rate-limiting.)*

3. **To require 2FA for SSH (SSH Key + TOTP code):**
Edit `/etc/pam.d/sshd`:
```bash
sudo nano /etc/pam.d/sshd
```
Add the following line at the bottom (and comment out `@include common-auth` or `auth substack password-auth` if you don't want it asking for your Linux password in addition to your SSH key and TOTP code):
```text
auth required pam_google_authenticator.so nullok
```
Then edit `/etc/ssh/sshd_config` to require both your public key and the PAM TOTP prompt:
```text
KbdInteractiveAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
```
Test and restart SSH:
```bash
sudo sshd -t && sudo systemctl restart sshd
```

4. **To require 2FA for `sudo`:**
If you want `sudo` to prompt for a TOTP code, add `auth required pam_google_authenticator.so` to `/etc/pam.d/sudo` (keep a root terminal open in another window to test before closing!).

# Set up a firewall

### 1. Install UFW

**Debian / Ubuntu:**
```bash
sudo apt update && sudo apt install ufw -y
```

**Fedora / RHEL:**
*(Note: Fedora comes with `firewalld` pre-installed. You can either use `firewall-cmd` or switch to `ufw`)*
```bash
sudo dnf install ufw -y
sudo systemctl disable --now firewalld
```

**Arch Linux:**
```bash
sudo pacman -S ufw
sudo systemctl enable --now ufw
```

After it is installed check the status:

```bash
sudo ufw status
```
At this point it should say it's inactive/disabled.
Before enabling it, open the ports needed to access the server—SSH at minimum:

```bash
sudo ufw allow ssh # or custom port
sudo ufw allow 22/tcp
```

### Rate-Limiting SSH with UFW

Instead of a standard `allow` rule, you can use UFW's `limit` rule for SSH. This denies connections from an IP address that has attempted to initiate 6 or more connections in the last 30 seconds, helping to mitigate brute-force attacks:

```bash
sudo ufw limit ssh
# Or if using a custom port:
sudo ufw limit 2222/tcp
```

Enable the firewall:
```bash
sudo ufw enable
```

Don't log out of this ssh session, start a new one to be safe.
If you are able to log in Great 👍 if not you will need to make sure that you have the right port open with the original session.

### Securing SSH with Tailscale and UFW

For even better security—especially on a public VPS—you can restrict SSH so it is **only** accessible over your private [Tailscale](https://tailscale.com/) network (Tailnet) and completely closed to the public internet.

1. Install Tailscale on the server:

**Debian / Ubuntu / Fedora:**
```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

**Arch Linux:**
```bash
sudo pacman -S tailscale
sudo systemctl enable --now tailscaled
```

2. Start Tailscale and authenticate your server:
```bash
sudo tailscale up
```
*(Optional: You can also use `sudo tailscale up --ssh` to let Tailscale manage SSH authentication directly via your Tailnet ACLs.)*

3. Verify your server's Tailscale IP address:
```bash
tailscale ip -4
```

4. Configure UFW to allow SSH only over the `tailscale0` network interface:
```bash
sudo ufw allow in on tailscale0 to any port 22 proto tcp
```
*(Alternatively, if you trust all devices on your Tailnet, you can allow all traffic on the Tailscale interface with `sudo ufw allow in on tailscale0`.)*

5. Test logging into the server using its Tailscale IP or MagicDNS hostname in a **new terminal window** before removing public access:
```bash
ssh username@tailscale-ip-address
```

6. Once confirmed working, remove the public SSH rules from UFW and reload:
```bash
sudo ufw delete allow ssh
sudo ufw delete allow 22/tcp
sudo ufw delete limit ssh
sudo ufw reload
sudo ufw status verbose
```
Now port 22 is completely hidden from the public internet and only reachable when connected to your Tailscale network.

### Docker + UFW Caveat (Preventing Accidental Port Exposure)

If you run **Docker** on your server, be careful: by default, Docker manipulates `iptables` directly and **bypasses UFW** whenever you publish a port with `-p 8080:80`. Even if UFW is set to deny incoming traffic on port `8080`, Docker will open it to the entire public internet!

To prevent this:

1. **Bind container ports to localhost (`127.0.0.1`) or your Tailscale IP** in your `docker run` or `docker-compose.yml` when they don't need direct public access (e.g., when sitting behind a reverse proxy or only accessed via Tailscale):
```yaml
ports:
  - "127.0.0.1:8080:80"
  # Or bind specifically to your server's Tailscale IP:
  # - "100.x.y.z:8080:80"
```

2. **Or use `ufw-docker`:**
Install [ufw-docker](https://github.com/chaifeng/ufw-docker), which modifies the `/etc/ufw/after.rules` configuration so that UFW remains in control of Docker container traffic:
```bash
sudo wget -O /usr/local/bin/ufw-docker https://github.com/chaifeng/ufw-docker/raw/master/ufw-docker
sudo chmod +x /usr/local/bin/ufw-docker
sudo ufw-docker install
sudo systemctl restart ufw
```
With `ufw-docker` installed, published Docker ports are blocked externally by default unless you explicitly allow them with `sudo ufw route allow proto tcp from any to any port 80`.

### Cloud Provider (NFOservers) / Hypervisor Firewall

In addition to running UFW inside the OS, you should block unwanted traffic **upstream** at the VPS provider or hypervisor level so malicious traffic is dropped before it even reaches your VM's virtual network card:

- **NFOservers VPS Firewall:** In the **NFOservers Control Panel**, go to your VDS/VPS management page and select the **Firewall** tab. You can add custom packet filter rules at NFO's network edge to drop all inbound TCP/UDP traffic to ports you don't publicly host, or restrict management ports (like SSH or game server RCON) to specific source IPs. NFO's upstream filtering is especially useful because it mitigates volumetric scans and floods before they consume your VM's CPU or bandwidth.
- **Homelab Hypervisor (e.g., Proxmox VE):** If running a local VM on Proxmox, enable the **Proxmox Firewall** at the Datacenter, Node, and VM levels with a default input policy of `DROP`, only permitting necessary LAN/Tailscale traffic.

# Intrusion Prevention: Fail2Ban and CrowdSec

## Option 1: Install, Configure, and Test Fail2Ban

If your SSH port (or other services) is exposed to the internet, **Fail2Ban** monitors log files for failed login attempts and automatically bans offending IP addresses using your firewall.

### 1. Install Fail2Ban

**Debian / Ubuntu:**
```bash
sudo apt update && sudo apt install fail2ban -y
```

**Fedora / RHEL:**
```bash
sudo dnf install fail2ban -y
```

**Arch Linux:**
```bash
sudo pacman -S fail2ban
```

### 2. Configure Fail2Ban

Fail2Ban reads `.conf` files first and overrides them with `.local` files. Never edit `/etc/fail2ban/jail.conf` directly, as package updates may overwrite it. Instead, copy it to `jail.local` or create a drop-in configuration file:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Under the `[DEFAULT]` section, you can adjust the ban settings:

```ini
[DEFAULT]
# Ban IP for 1 hour (or increase to 1d, 1w, etc.)
bantime  = 1h

# Time window in which maxretry failures trigger a ban
findtime  = 10m

# Number of failures before an IP is banned
maxretry = 5

# Ignore localhost and your trusted subnet / Tailscale range
ignoreip = 127.0.0.1/8 ::1 100.64.0.0/10
```

Next, scroll down to the `[sshd]` section (or create `/etc/fail2ban/jail.d/sshd.local`) and ensure it is enabled. On modern Debian/Ubuntu, Fedora, and Arch systems using `systemd` journals, set `backend = systemd`:

```ini
[sshd]
enabled = true
port    = ssh
logpath = %(sshd_log)s
backend = systemd
```

Enable and restart the Fail2Ban service:

```bash
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban
```

### 3. Test and Verify Fail2Ban

Check that the Fail2Ban service is running and that the `sshd` jail is active:

```bash
sudo systemctl status fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

You should see output showing the number of currently failed and banned IPs for `sshd`.

**To test that banning works:**
1. From another machine (not listed in `ignoreip`), intentionally fail an SSH login several times (or temporarily set `PasswordAuthentication yes` on a test VM and enter a wrong password `maxretry` times):
```bash
ssh -o PubkeyAuthentication=no invaliduser@server-ip-address
```
2. Check the jail status again on your server:
```bash
sudo fail2ban-client status sshd
```
You will see the test machine's IP listed under `Banned IP list`.

**How to manually unban an IP address:**
If you accidentally lock yourself or a legitimate user out, unban the IP with:

```bash
sudo fail2ban-client set sshd unbanip <IP-ADDRESS>
```

## Option 2: CrowdSec (Modern Collaborative Alternative to Fail2Ban)

[CrowdSec](https://www.crowdsec.net/) is a modern, open-source intrusion prevention system that acts like a crowd-sourced Fail2Ban. In addition to detecting local attacks on SSH, Nginx, Traefik, Caddy, and Docker logs, it shares and receives curated community blocklists of known malicious IPs so attackers are blocked before they even touch your server.

### 1. Install CrowdSec and the Firewall Bouncer

**Debian / Ubuntu:**
```bash
curl -s https://install.crowdsec.net | sudo sh
sudo apt update
sudo apt install crowdsec crowdsec-firewall-bouncer-nftables -y
# (Or crowdsec-firewall-bouncer-iptables if using legacy iptables)
```

**Fedora / RHEL:**
```bash
curl -s https://install.crowdsec.net | sudo sh
sudo dnf install crowdsec crowdsec-firewall-bouncer-nftables -y
```

**Arch Linux (via AUR):**
```bash
yay -S crowdsec crowdsec-firewall-bouncer
sudo systemctl enable --now crowdsec
sudo systemctl enable --now crowdsec-firewall-bouncer
```

### 2. Verify and Manage CrowdSec with `cscli`

During installation, CrowdSec automatically detects running services (like `sshd`, `nginx`, etc.) and installs the matching log parsers and scenarios.

Check installed collections and active bouncers:
```bash
sudo cscli collections list
sudo cscli bouncers list
```

View log parsing metrics to ensure SSH logs are being monitored:
```bash
sudo cscli metrics
```

View active bans (decisions) and recent alerts:
```bash
sudo cscli decisions list
sudo cscli alerts list
```

**Test or manually ban/unban an IP:**
```bash
# Manually ban an IP for 4 hours to test the bouncer
sudo cscli decisions add --ip 198.51.100.25 --duration 4h --reason "manual test"

# Remove a ban
sudo cscli decisions delete --ip 198.51.100.25
```

# Enable automatic updates

In a non-production environment, automatic updates are convenient, though an update can potentially break something—so if the system is mission critical, enable at your own risk assessment and test updates on a staging server first.
(My opinion anyway)

### Debian / Ubuntu (`unattended-upgrades`)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Verify the timer is active:
```bash
systemctl status apt-daily-upgrade.timer
```

### Fedora / RHEL (`dnf-automatic`)

```bash
sudo dnf install dnf-automatic -y
sudo nano /etc/dnf/automatic.conf
```
In `/etc/dnf/automatic.conf`, set `apply_updates = yes` (and optionally set `upgrade_type = security` if you only want security patches applied automatically). Then enable the systemd timer:

```bash
sudo systemctl enable --now dnf-automatic.timer
```

### Arch Linux

On a rolling-release distribution like Arch Linux, unattended full-system upgrades (`pacman -Syu --noconfirm`) are **not recommended** because manual intervention or `.pacnew` merges are occasionally required. Instead, install `pacman-contrib` and check for pending updates regularly (or send notifications) using `checkupdates`:

```bash
sudo pacman -S pacman-contrib
checkupdates
```

That's all for now, I will update this document with any additional details or updates.

