---
title: TeamSpeak 3 and 6 Server Permissions & Security Configuration
author: Mainedan
date: 2026-10-08 04:00:00 -0400
categories: [Teamspeak, Documentation]
tags: [teamspeak, documentation, permissions]
---

# TeamSpeak 3 and 6 Server Permissions & Security Guide

This guide walks through the practical steps for securing a TeamSpeak 3 or TeamSpeak 6 server with the advanced permission system. The goal is simple: keep the real owner in full control, limit secondary admin power, lock down private channels, and prevent common privilege escalation mistakes.

TeamSpeak 6 uses the same permission model as TeamSpeak 3 in many cases, but the client and interface continue to evolve. Treat this as a strong baseline, then verify any menu names or permission labels inside your own server before saving changes.

{% include embed/youtube.html id='CDzk2KbYcVk?si=okNxQvH_N758oqnc' %}
📺 [Watch Video](https://youtu.be/CDzk2KbYcVk?si=okNxQvH_N758oqnc)

---

## 1. Enable the Advanced Permission System

This step is only required when using the TeamSpeak 3 client.

1. Open TeamSpeak 3.
2. Go to **Settings** > **Options**.
3. Under the **Application** tab, enable **Advanced permission system**.
4. Click **Apply** or **OK**.

![Advanced Permission System](/assets/images/ts/AdvancedPermissionSystem.png)

---

## 2. Build a Safe Administrative Hierarchy

By default, the **Server Admin** group has full power at level `75`. If you assign that group to other staff members, they can change server settings, remove your ownership privileges, or effectively hijack the server.

The safest pattern is:

- Keep the original **Server Admin** group as the true owner group and leave it at `75`.
- Create a secondary admin group with reduced power at `70`.
- Only assign trusted staff to the reduced-power admin group.

### Step 2.1: Duplicate the Server Admin Group

This is currently only possible in the TeamSpeak 3 client.

1. Open **Permissions** > **Server Groups**.
2. Right-click **Server Admin** and choose **Copy**.
3. Name the new group **Admin** or **Sub-Admin**.
4. Give the original owner group a unique icon so it is visually obvious that it is the real owner group.

![Duplicate the Server Admin Group](/assets/images/ts/server-admin-copy.png)

### Step 2.2: Reduce Secondary Admin Power

Use the copied **Admin** group and lower the key values to `70` for the following areas.

#### Group permissions

In **Group** > **Modify**:

- **Group Modify Power** = `70`
- **Group Member Add Power** = `70`
- **Group Member Remove Power** = `70`
- **Permission Modify Power** = `70`

#### Channel permissions

In **Channel** > **Modify**, **Access**, and **Delete**:

- **Channel Modify Power** = `70`
- **Channel Delete Power** = `70`
- **Channel Permission Modify Power** = `70`
- **Channel Join Power** = `70`
- **Channel Subscribe Power** = `70`
- **Channel Description View Power** = `70`

#### Virtual server settings

Under **Virtual Server** > **Settings**, remove or disable the ability to change:

- **Modify Virtual Server Max Clients**
- **Modify Virtual Server Name**
- **Modify Virtual Server Reserved Slots**
- Default group settings
- Default admin group settings
- Anti-flood settings
- Host message, banner, and button settings
- Port settings
- Log settings
- Transfer quotas and transfer settings

#### Client permissions

Under **Client** > **Administration** and **Basic**:

- **Client Kick From Server Power** = `70` with **Needed Client Kick From Server Power** = `75`
- **Client Kick From Channel Power** = `70` with **Needed Client Kick From Channel Power** = `75`
- **Client Ban From Server Power** = `70` with **Needed Client Ban From Server Power** = `75`
- **Client Move Power** = `70` with **Needed Client Move Power** = `75`
- **Client Complain Power** = `70` with **Needed Client Complain Power** = `75`
- **Private Text Message Power** = `70`
- **Client Talk Power** = `70`
- **Client Poke Power** = `70`
- **Needed Poke Power** = `70` or `75`, depending on whether you want guests or only owners/admins to be able to poke
- Remove **Delete Own Ban Rules** and **Delete All Ban Rules**
- Remove **Client Permission Modify Power** and **Skip Client Group & Channel Permissions**

#### Limited feature control

For secondary admins, only grant small, safe control points:

- **Priority Speaker** = `70`
- **Icon ID** = `70`
- **Group Sort ID** = `70`

This keeps admins useful without giving them the ability to rebuild the server hierarchy, take ownership, or lock out the real owner.

### Step 2.3: Remove Dangerous Bypass Flags

Under **Client**:

- Disable **Skip Channel Group & Channel Permissions** (`b_client_skip_channelgroup_permissions`)

Under **Channel** > **Access**:

- Disable **Ignore Channel Passwords** (`b_channel_join_ignore_password`)

This matters because secondary admins are then forced to respect channel passwords and cannot override private channels.

### Step 2.4: Block Backdoor Access and Privilege Key Abuse

Under **Virtual Server Administration**:

- Disable **Create New Privilege Key** (`b_virtualserver_token_add`)
- Disable **View List of Available Privilege Keys** (`b_virtualserver_token_list`)

Under **Group Information**:

- Disable **View List of Client Permissions** (`b_client_permission_list`)
- Restrict or disable direct client permission editing (`b_client_permission_modify`)

The goal is to stop admins from creating hidden privilege tokens or self-granting unauthorized permissions.

### Step 2.5: Set Moderation Limits

Keep moderation power available without allowing destructive abuse:

- Keep **Kick Power**, **Ban Power**, **Poke Power**, **Talk Power**, and **Private Message Power** at `70`
- Set a maximum ban duration, such as `86400` seconds (24 hours), so admins cannot accidentally create permanent bans

### Step 2.6: Prevent Emergency Owner Lockout

As the main server owner:

1. Right-click your own name and open **Permissions** > **Client Permissions**.
2. Give yourself explicit `75` power for:
   - **Group Member Add Power**
   - **Group Member Remove Power**
   - **Group Modify Power**

This ensures that even if you accidentally remove your own **Server Admin** group, you still have enough authority to restore it.

![Emergency Owner Lockout Prevention](/assets/images/ts/EmergencyOwnerLockoutPrevention.png)

---

## 3. Pre-Configure Channel Group Permissions

A secure channel setup starts with real channel groups. This prevents accidental access issues later when you lock channels down.

1. Open **Permissions** > **Channel Groups**.
2. Select each real, non-default channel group such as **Voice**, **Operator**, or **Channel Admin**.
3. Exclude the default **Guest** group.
4. Under **Channel** > **Access**, enable all three join permissions:
   - `b_channel_join_permanent`
   - `b_channel_join_semi_permanent`
   - `b_channel_join_temporary`
5. Set **Channel Join Power** = `50` and **Channel Subscribe Power** = `50` for the assigned groups.

This gives legitimate channel groups enough access to work normally while still preserving a hardened channel hierarchy.

![Channel Group Permissions](/assets/images/ts/ChannelGroupPermissions.png)

---

## 4. Lock Down Channels and Hide Occupants

This is the most effective way to make a channel effectively private and invisible to unauthorized users.

### Step 4.1: Revoke Channel Join and Subscribe Power

1. Right-click the target channel and choose **Channel Permissions**.
2. Under **Channel** > **Access**:
   - Disable **Join Permanent**
   - Disable **Join Semi-Permanent**
   - Disable **Join Temporary**
   - Set **Channel Join Power** = `0`
   - Set **Channel Subscribe Power** = `0`

This creates a zero-power lock at the channel level. Even if a user has a server-group permission, the channel will still block them unless they are explicitly granted access.

![User Channel Permissions](/assets/images/ts/user_channel.png)

### Step 4.2: Set High Required Channel Powers

Set the needed powers high enough that ordinary users cannot enter or see the channel.

#### TeamSpeak 3 client

Right-click the channel and open **Edit Channel** or the channel permissions panel:

- **Needed Channel Join Power** = `50` or `75` for owner-only channels
- **Needed Channel Subscribe Power** = `50` or `75` for owner-only channels
- **Needed Channel Description View Power** = `50` or `75` for owner-only channels
- **Needed Channel Modify Power** = `70` or `75` for owner-only channels
- **Needed Channel Delete Power** = `70` or `75` for owner-only channels

#### TeamSpeak 6 client

Use the same concept in the permissions interface:

- Add **Needed Channel Join Power** = `50` or `75`
- Add **Needed Channel Subscribe Power** = `50` or `75`
- Add **Needed Channel Description View Power** = `50` or `75`
- Add **Needed Channel Modify Power** = `70` or `75`
- Add **Needed Channel Delete Power** = `70` or `75`

The result is that outside users generally see the channel as empty, and occupant lists and movement notifications remain hidden.

---

## 5. Grant Access to Specific Members

If you want to give a single user access to a locked channel, do it in a controlled way.

1. Drag the user into the secure channel.
2. Right-click the user and choose **Set Channel Group**.
3. Assign a non-guest group such as **Voice**.

Channel access works on a clear hierarchy:

**Channel Group Permissions > Channel Permissions > Server Group Permissions**

Because the channel group has `50` join and subscribe power, it overrides the channel lock and satisfies the required channel permissions.

---

## 6. Operational Server Settings and Privacy

A secure TeamSpeak server is not just about groups and channels. You also need to harden some server-level settings.

### Lock down guest whispers

Under **Permissions** > **Server Groups** > **Guest**:

- Set **Client Whisper Power** = `-100`

This prevents spammers and new users from whispering audio across the server.

### Use reserved slots

Right-click the server and open **Edit Virtual Server**:

- Set **Reserved Slots** (for example `3`)
- Enable `b_client_use_reserved_slot` on admin groups so staff can still connect when the server is full

### Protect the host message

Use **Modal Log** or **None** in the virtual server settings.

Avoid **Modal Quit** because it can disconnect clients in a loop.

### Disable public server listing if needed

Turn off **Enable reporting to server list** if you do not want your server to appear publicly.

### Audit logging

Turn on server logging and monitor:

- admin actions
- permission changes
- channel modifications
- connection events

### Poke settings

**Needed Poke Power** should be tuned to your comfort level:

- Set it to `50` if you want normal users to be able to poke each other under normal conditions
- Set it to `70` for a stricter admin-only moderation setup
- Keep it at `75` for the owner/admin tier if you want to prevent lower-power groups from poking higher-tier users

---

## 7. Recommended Admin Baseline

If you want a clean, practical starting point, this is the setup I recommend:

- Keep the original **Server Admin** group at `75`
- Create a new **Admin** group and set it to `70`
- Remove dangerous bypass flags and token creation permissions
- Restrict direct client permission editing
- Keep ban and kick powers low enough for moderation but not enough to hijack the server
- Lock private channels with zero join power and higher required power values
- Give access only through dedicated channel groups, not broad server-wide permissions

The most important principle is this: keep the true owner group at full power, and make every other admin group a controlled subset of it.

---

## Final Notes

This setup is meant to prevent the most common TeamSpeak failures: accidental admin removal, admin hijack attempts, hidden privilege token abuse, and insecure channel access.

If you are running a public or semi-public server, do not rely on default permissions alone. The safer configuration is to start locked down and grant access only where you truly need it.

With the right hierarchy, private channels, and permission limits, your TeamSpeak server remains usable for staff without becoming vulnerable to administrative abuse.
