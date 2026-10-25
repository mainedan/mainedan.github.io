---
title: Securing a Debian VPS with Tailscale, UFW, Fail2Ban, and SSH Tunneling (Ports 22 & 443)
author: Mainedan
date: 2026-10-25 09:00:00 -0400
categories: [Linux, Documentation]
tags: [homelab, documentation, linux, debian, vps, security, tailscale, ufw, fail2ban, ssh-tunnel]
---

# Securing a Debian VPS with Tailscale, UFW, Fail2Ban, and SSH Tunneling

When setting up a Debian VPS, you want locked-down security where **only you** can authenticate, while still maintaining the flexibility to connect and run **SSH tunnels** from anywhere—even on restrictive public Wi-Fi, hotel, or corporate networks that block Tailscale (UDP) or standard SSH (`22/tcp`).

By combining **Tailscale**, **OpenSSH on ports `22` and `443`**, **UFW (Uncomplicated Firewall)** with rate-limiting, and **Fail2Ban**, you get:

- **Private Mesh Access:** Full access over your encrypted `tailscale0` interface.
- **SSH Tunneling Anywhere (`22` & `443`):** Public key-only SSH listening on both port `22` and port `443` (HTTPS port, which is almost never blocked by outbound firewalls) for SOCKS5 proxying and port forwarding.
- **Layered Firewall & Brute-Force Protection:** UFW drops all unauthorized ports, rate-limits connection attempts on ports `22` and `443`, and Fail2Ban automatically bans scanners across both ports.

---

## Prerequisites & Lockout Prevention

This guide assumes you have already completed the basic initial server setup (see [Securing a Linux server](/posts/secure-your-server/)):

- Your Debian VPS is updated (`sudo apt update && sudo apt upgrade -y`).
- You have created a non-root user with `sudo` privileges.
- You have uploaded your `ed25519` SSH public key (`~/.ssh/authorized_keys`).

> **Important — Keep Your Current SSH Session Open:**
> Whenever editing `/etc/ssh/sshd_config` or enabling UFW on a remote VPS, **never close your active SSH terminal** until you have tested logging in from a **second terminal window**.

---

## Step 1: Configure `sshd` for Ports 22 & 443 and Enable SSH Tunneling

First, we will configure OpenSSH to listen on both port `22` and port `443`, lock authentication strictly to your SSH key, and enable TCP forwarding (`AllowTcpForwarding yes`) so SSH tunneling works.

> **Note on Port 443:** Make sure no web server (like Nginx or Apache) is currently bound to port `443` before restarting `ssh`, otherwise `sshd` will fail to bind to `443`. *(If you ever want to run both an HTTPS web server and SSH on port `443` simultaneously, see the note on `sslh` in Step 3.)*

### 1. Back Up and Edit `/etc/ssh/sshd_config`

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config
```

Update or add the following lines:

```text
# Listen on both standard SSH (22) and HTTPS (443) for firewall bypass
Port 22
Port 443

# Disable root login and all password/interactive authentication
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
PubkeyAuthentication yes

# Limit authentication attempts and explicitly whitelist your user
MaxAuthTries 3
AllowUsers username

# IMPORTANT: Enable TCP forwarding so SSH tunneling (-D, -L, -R) works!
AllowTcpForwarding yes
GatewayPorts no
X11Forwarding no
```

*(Note: If you previously followed a hardening guide that set `AllowTcpForwarding no`, make sure to change it to `AllowTcpForwarding yes` or your SOCKS5 and local port tunnels will be rejected by the server.)*

### 2. Test Syntax and Restart SSH

Always verify your configuration syntax before restarting the SSH daemon:

```bash
sudo sshd -t
```

If there is no output, the configuration is valid. Restart the SSH service and verify it is listening on both `:22` and `:443`:

```bash
sudo systemctl restart ssh
sudo ss -tlnp | grep sshd
```

You should see `sshd` listening on both `0.0.0.0:22` and `0.0.0.0:443` (as well as `[::]:22` and `[::]:443`).

---

## Step 2: Set Up Tailscale and Tailscale SSH

Installing [Tailscale](https://tailscale.com/) creates an encrypted WireGuard mesh network (`tailscale0`) between your personal devices and your Debian VPS.

### 1. Install Tailscale on Debian

Run the official Tailscale install script:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

### 2. Start Tailscale with SSH Enabled

Bring the Tailscale interface up and enable **Tailscale SSH**:

```bash
sudo tailscale up --ssh
```

Follow the authentication link printed in the terminal to log into your Tailscale account and authorize the VPS.

### 3. Check Your Server's Tailscale IP

Verify your VPS's private Tailscale IPv4 address (in the `100.64.0.0/10` range):

```bash
tailscale ip -4
tailscale status
```

With this setup, you can connect via:
- **Tailscale IP (`100.x.y.z`):** Authenticates via your Tailnet identity (`--ssh`) or your SSH key, completely isolated from the public internet.
- **Public IP on Port `22` or Port `443`:** Authenticates strictly via your `ed25519` SSH key when you are on a machine or network where Tailscale cannot connect directly.

---

## Step 3: Using SSH Tunneling on Port 22 and Port 443

With `Port 22`, `Port 443`, and `AllowTcpForwarding yes` enabled, you can configure your local `~/.ssh/config` to make connecting and tunneling effortless.

### 1. Configure Local `~/.ssh/config` Profiles

On your **local (client) machine**, edit `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Add profiles for Tailscale, Public Port 22, Public Port 443, and a ready-to-use SOCKS5 proxy tunnel:

```text
# 1. Primary connection over Tailscale
Host vps-ts
    HostName 100.x.y.z
    User username
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

# 2. Public SSH over standard Port 22
Host vps-22
    HostName your-vps-public-ip
    User username
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

# 3. Public SSH over Port 443 (bypasses strict outbound firewalls)
Host vps-443
    HostName your-vps-public-ip
    User username
    Port 443
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
    # Optional: Start a local SOCKS5 proxy on 127.0.0.1:1080 automatically
    # DynamicForward 1080
```

Set proper permissions on your local config:

```bash
chmod 600 ~/.ssh/config
```

### 2. Common SSH Tunneling Commands

#### A. Dynamic SOCKS5 Proxy (`ssh -D`) — Route Browser / App Traffic Through the VPS
Creates a local SOCKS5 proxy on `127.0.0.1:1080` that tunnels all traffic through your Debian VPS (great for restrictive Wi-Fi where only port 443 is open):

```bash
# Tunnel over Port 443:
ssh -D 1080 -N -C -p 443 username@your-vps-public-ip
# Or using the ~/.ssh/config alias:
ssh -D 1080 -N -C vps-443

# Tunnel over Port 22:
ssh -D 1080 -N -C vps-22
```
- `-D 1080`: Opens a local SOCKS5 proxy on port `1080`. Configure your browser (e.g., Firefox Network Settings -> Manual proxy configuration -> SOCKS Host `127.0.0.1`, Port `1080`, SOCKS v5, and check *Proxy DNS when using SOCKS v5*).
- `-N`: Do not execute a remote shell command (just forward ports).
- `-C`: Enable compression (add `-f` if you want it to run in the background).

#### B. Local Port Forwarding (`ssh -L`) — Access a Private Service on the VPS
Forwards a port from your local machine (`127.0.0.1:8080`) through the encrypted SSH tunnel to a service bound only to `127.0.0.1:8080` on the VPS:

```bash
# Over Port 443:
ssh -L 127.0.0.1:8080:127.0.0.1:8080 -N vps-443

# Over Port 22 (or Tailscale):
ssh -L 127.0.0.1:8080:127.0.0.1:8080 -N vps-22
```
Then open `http://localhost:8080` in your local browser.

#### C. Remote Port Forwarding (`ssh -R`) — Expose a Local Service to the VPS
Forwards a port on the VPS (`127.0.0.1:9000`) back to a service running on your local machine (`127.0.0.1:3000`):

```bash
ssh -R 127.0.0.1:9000:127.0.0.1:3000 -N vps-443
```

---

## Step 4: Set Up the UFW Firewall

Now we will configure **UFW** to block all unwanted incoming connections while:
1. Allowing all traffic on your private `tailscale0` interface.
2. Allowing UDP `41641` for direct peer-to-peer Tailscale connections.
3. Allowing and **rate-limiting** TCP ports `22` and `443` (`ufw limit`) so brute-force connection floods are throttled at the firewall level.

### 1. Install UFW and Set Default Policies

```bash
sudo apt update && sudo apt install ufw -y
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 2. Allow Tailscale Interface and Direct WireGuard Port

```bash
sudo ufw allow in on tailscale0 comment 'Allow all traffic over Tailscale'
sudo ufw allow 41641/udp comment 'Tailscale direct peer-to-peer'
```

*(Note: If you only want SSH ports accessible on `tailscale0` instead of all ports, use `sudo ufw allow in on tailscale0 to any port 22 proto tcp`.)*

### 3. Allow and Rate-Limit SSH on Ports 22 and 443

Using `ufw limit` instead of a plain `ufw allow` automatically denies connections from any IP address that attempts to initiate 6 or more connections within 30 seconds:

```bash
sudo ufw limit 22/tcp comment 'SSH and tunneling on port 22 (rate-limited)'
sudo ufw limit 443/tcp comment 'SSH and tunneling on port 443 (rate-limited)'
```

*(Note: If you had an older generic `allow ssh` or `allow 22/tcp` rule, you can remove the non-rate-limited duplicate with `sudo ufw delete allow 22/tcp`.)*

### 4. Enable UFW and Verify Rules

Enable the firewall and check the active rules:

```bash
sudo ufw enable
sudo ufw reload
sudo ufw status verbose
```

Expected output:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
Anywhere on tailscale0     ALLOW IN    Anywhere                   # Allow all traffic over Tailscale
41641/udp                  ALLOW IN    Anywhere                   # Tailscale direct peer-to-peer
22/tcp                     LIMIT IN    Anywhere                   # SSH and tunneling on port 22 (rate-limited)
443/tcp                    LIMIT IN    Anywhere                   # SSH and tunneling on port 443 (rate-limited)
Anywhere (v6) on tailscale0 ALLOW IN   Anywhere (v6)              # Allow all traffic over Tailscale
41641/udp (v6)             LIMIT IN    Anywhere (v6)              # Tailscale direct peer-to-peer
22/tcp (v6)                LIMIT IN    Anywhere (v6)              # SSH and tunneling on port 22 (rate-limited)
443/tcp (v6)               LIMIT IN    Anywhere (v6)              # SSH and tunneling on port 443 (rate-limited)
```

Before closing your original terminal, test logging in over Tailscale (`ssh vps-ts`), Port 22 (`ssh vps-22`), and Port 443 (`ssh vps-443`) in a new terminal window! 👍

### 5. Selectively Opening Additional Public Service Ports (Optional)

If you later host other public services on this VPS, open only the specific ports needed:

**TeamSpeak 3 / 6 Server:**
```bash
sudo ufw allow 9987/udp comment 'TeamSpeak Voice'
sudo ufw allow 30033/tcp comment 'TeamSpeak File Transfer'
```

**Web Server + SSH Sharing Port 443 (`sslh` Tip):**
Because `sshd` is listening on `443/tcp`, a web server like Nginx cannot bind directly to `0.0.0.0:443` at the same time. If you later run an HTTPS website on this VPS alongside SSH on port 443, install `sslh` (`sudo apt install sslh`), have Nginx listen on `127.0.0.1:8443` and `sshd` on `22`, and let `sslh` listen on `443` to automatically route SSH traffic to `127.0.0.1:22` and HTTPS traffic to `127.0.0.1:8443`.

**Docker + UFW Caveat:**
If you use Docker, remember that `-p 8080:80` bypasses UFW by default. Always bind private container ports to localhost (`- "127.0.0.1:8080:80"`) or your Tailscale IP (`- "100.x.y.z:8080:80"`), which you can then securely reach using your SSH local port forward (`ssh -L`) or Tailscale!

---

## Step 5: Configure Fail2Ban to Protect Ports 22 & 443 via UFW

Because ports `22` and `443` are reachable on the public internet for SSH tunneling, **Fail2Ban** is critical. It monitors the systemd journal for failed SSH handshakes and key rejections across **both** ports `22` and `443` and inserts a UFW `REJECT` rule to block the attacker's IP across the entire server.

### 1. Install Fail2Ban and the Systemd Backend

On modern Debian (Debian 12 Bookworm / Debian 13 Trixie), install `fail2ban` along with `python3-systemd` so Fail2Ban can read `systemd-journald` logs directly:

```bash
sudo apt update && sudo apt install fail2ban python3-systemd -y
```

### 2. Configure `/etc/fail2ban/jail.local`

Create or edit `/etc/fail2ban/jail.local`:

```bash
sudo nano /etc/fail2ban/jail.local
```

Add the following configuration. Notice that:
- `banaction = ufw` tells Fail2Ban to block offending IPs directly inside UFW.
- `ignoreip` whitelists localhost and your entire **Tailscale subnet (`100.64.0.0/10`)** so your Tailnet devices are never banned.
- `[sshd]` is configured with `port = 22,443` so failed attempts on **either** port 22 or port 443 count toward a ban.

```ini
[DEFAULT]
# Ban offending IPs for 1 hour (or increase to 1d, 1w, etc.)
bantime  = 1h

# Time window in which maxretry failures trigger a ban
findtime = 10m

# Number of failures before an IP is banned
maxretry = 5

# Read logs from systemd journal on modern Debian
backend  = systemd

# Use UFW to enforce bans across all ports when an attacker triggers a jail
banaction = ufw[port=""]
banaction_allports = ufw[port=""]

# Whitelist localhost and the entire Tailscale CGNAT range
ignoreip = 127.0.0.1/8 ::1 100.64.0.0/10

# ==========================================
# SSH Jail (Monitors BOTH Port 22 and Port 443)
# ==========================================
[sshd]
enabled  = true
port     = 22,443
protocol = tcp
mode     = aggressive

# ==========================================
# Optional: Public Web / Reverse Proxy Jails
# (Uncomment if Nginx is installed and publicly exposed)
# ==========================================
#[nginx-http-auth]
#enabled  = true
#port     = http,https
#backend  = auto
#logpath  = /var/log/nginx/error.log

#[nginx-botsearch]
#enabled  = true
#port     = http,https
#backend  = auto
#logpath  = /var/log/nginx/access.log
#maxretry = 2

#[nginx-badbots]
#enabled  = true
#port     = http,https
#backend  = auto
#logpath  = /var/log/nginx/access.log
#maxretry = 2
```

*(Why `banaction = ufw[port=""]`? When monitoring multiple ports like `22,443`, passing an empty port parameter instructs UFW to run `ufw insert 1 reject from <attacker-ip> to any`, blocking the attacker from **all** ports on your VPS rather than just one port, while avoiding UFW multi-port syntax issues.)*

### 3. Enable and Restart Fail2Ban

```bash
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban
```

### 4. Verify and Test Fail2Ban with UFW

Verify that Fail2Ban is running and monitoring `sshd`:

```bash
sudo systemctl status fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

**Test Banning an IP and Verify in UFW:**

Simulate banning a test IP (`198.51.100.25`):

```bash
sudo fail2ban-client set sshd banip 198.51.100.25
```

Check both Fail2Ban and UFW status:

```bash
sudo fail2ban-client status sshd
sudo ufw status verbose
```

You will see `198.51.100.25` in the `Banned IP list` and a `REJECT IN` rule at the top of `sudo ufw status verbose` blocking `198.51.100.25` from reaching any port (`22`, `443`, etc.) on your VPS.

**Remove the Test Ban (or Unban an IP):**

```bash
sudo fail2ban-client set sshd unbanip 198.51.100.25
```

---

## Summary Checklist

- [x] **`sshd` configured** to listen on both `Port 22` and `Port 443`, with `PasswordAuthentication no`, `PermitRootLogin no`, and `AllowTcpForwarding yes` for SSH tunneling (`-D`, `-L`, `-R`).
- [x] **Tailscale installed** (`sudo tailscale up --ssh`) for private mesh access over `tailscale0`.
- [x] **UFW enabled** with `default deny incoming`, allowing `tailscale0`, `41641/udp`, and rate-limited (`LIMIT IN`) public SSH on `22/tcp` and `443/tcp`.
- [x] **Fail2Ban running** with `backend = systemd`, `banaction = ufw[port=""]`, Tailscale `100.64.0.0/10` whitelisted in `ignoreip`, and `[sshd]` protecting both ports `22` and `443`.
