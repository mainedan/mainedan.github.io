---
title: TeamSpeak 3 and 6 Server Permissions & Security Configuration
author: Mainedan
date: 2026-09-30 04:00:00 -400
categories: [Teamspeak, Documentation]
tags: [teamspeak,documentation,permissions]
---

--------------------------------------------------------------------------------

# TeamSpeak 3 and 6 Server Permissions & Security Guide

---

- This guide outlines the step-by-step procedure for configuring a TeamSpeak 3 or 6 server using the Advanced Permission System. It covers establishing a secure administrative hierarchy to prevent server hijacking, locking down channels, hiding channel occupants, configuring granular channel access, and tuning operational server settings. Teamspeak 6 uses the same settings that teamspeak server 3 uses, but teamspeak 6 is still in active developpment, so some things can change.

Used for the guide

{% include embed/youtube.html id='CDzk2KbYcVk?si=okNxQvH_N758oqnc' %}
📺 [Watch Video](https://youtu.be/CDzk2KbYcVk?si=okNxQvH_N758oqnc)

---

## 1. Enable the Advanced Permission System (Only required if using TeamSpeak 3 Client)

1. Open TeamSpeak 3.
2. Navigate to **Settings** > **Options**.
3. Under the **Application** tab, locate and check **Advanced permission system**.
4. Click **Apply** or **OK**.

![Advanced Permission System](/assets/images/ts/AdvancedPermissionSystem.png)

## 2. Establish a Safe Administrative Hierarchy (Prevent Hijacking)

By default, the *Server Admin* group has absolute power (`75`). Giving other staff members this default group allows them to modify server settings, take away the owner's admin status, or hijack ownership.

### Step 2.1: Duplicate the Server Admin Group (currently only possible in the TeamSpeak 3 client)

1. Go to **Permissions** > **Server Groups**.
2. Right-click **Server Admin** and select **Copy**. Name the new group **Admin** or **Sub-Admin**.
3. Give the original owner group a distinct icon so it is visually obvious who holds full server ownership.

![Duplicate the Server Admin Group](/assets/images/ts/server-admin-copy.png)

### Step 2.2: Restrict Secondary Admin Power Levels

Create a reduced-permission admin group and keep the true owner group at `75`.

Use the copied **Admin** group and lower the key permission values to `70` for the following areas:

1. **Group Permissions** (`Group` > `Modify`):
   - **Group Modify Power** = `70`
   - **Group Member Add Power** = `70`
   - **Group Member Remove Power** = `70`
   - **Permission Modify Power** = `70`

2. **Channel Permissions** (`Channel` > `Modify`, `Access`, `Delete`):
   - **Channel Modify Power** = `70`
   - **Channel Delete Power** = `70`
   - **Channel Permission Modify Power** = `70`
   - **Channel Join Power** = `70`
   - **Channel Subscribe Power** = `70`
   - **Channel Description View Power** = `70`

3. **Virtual Server** (`Virtual Server` > `Settings`):
   - Remove or disable **Modify Virtual Server Max Clients**.
   - Remove or disable **Modify Virtual Server Name**.
   - Remove or disable **Modify Virtual Server Reserved Slots**.
   - Remove or disable the default group, default admin group, and related server-setting modifications, including anti-flood, host message/banner/button, port, log settings, and transfer quotas/settings.

4. **Client** (`Client` > `Administration` and `Basic`):
   - **Client Kick From Server Power** = `70` with **Needed Client Kick From Server Power** = `75`
   - **Client Kick From Channel Power** = `70` with **Needed Client Kick From Channel Power** = `75`
   - **Client Ban From Server Power** = `70` with **Needed Client Ban From Server Power** = `75`
   - **Client Move Power** = `70` with **Needed Client Move Power** = `75`
   - **Client Complain Power** = `70` with **Needed Client Complain Power** = `75`
   - **Private Text Message Power** = `70`
   - **Client Talk Power** = `70`
   - **Client Poke Power** = `70`
   - **Needed Poke Power** = `70` or `75` depending on whether you want guests or only owners/admins to be able to poke.
   - Remove **Delete Own Ban Rules** and **Delete All Ban Rules**.
   - Remove **Client Permission Modify Power** and **Skip Client Group & Channel Permissions**.

5. **Grant only limited feature control**:
   - Set **Priority Speaker** grant to `70`
   - Set **Icon ID** grant to `70`
   - Set **Group Sort ID** grant to `70`

This keeps secondary admins useful without giving them the power to change server ownership, rebuild admin groups, or lock out the real owner.

![ChannelPermissions](/assets/images/ts/ChannelPermissions.png)

![Restrict Secondary Admin Power Levels](/assets/images/ts/Restrict Secondary Admin Power Levels.png)

![Virtual Server](/assets/images/ts/VirtualServer.png)

### Step 2.3: Revoke Dangerous Bypass Flags

1. Under **Client**:
   - Disable **Skip Channel Group & Channel Permissions** (`b_client_skip_channelgroup_permissions`).
2. Under **Channel** > **Access**:
   - Disable **Ignore Channel Passwords** (`b_channel_join_ignore_password`).
   - *Security result:* Secondary admins are forced to respect channel passwords and cannot barge into private channels.

### Step 2.4: Block Backdoor Access & Privilege Key Exploits

1. Under **Virtual Server Administration**:
   - Disable **Create New Privilege Key** (`b_virtualserver_token_add`)
   - Disable **View List of Available Privilege Keys** (`b_virtualserver_token_list`)
2. Under **Group Information**:
   - Disable **View List of Client Permissions** (`b_client_permission_list`)
   - Restrict or disable direct client permission editing (`b_client_permission_modify`)
   - *Security result:* Admins cannot generate hidden privilege tokens or grant themselves unauthorized permissions.

### Step 2.5: Configure Moderation Limits

Keep moderation power available without allowing destructive abuse:

- Set **Kick Power**, **Ban Power**, **Poke Power**, **Talk Power**, and **Private Message Power** to `70`.
- Set a maximum ban duration (for example `86400` seconds / 24 hours) so admins cannot issue permanent bans by mistake.

### Step 2.6: Emergency Owner Lockout Prevention

1. As the main server owner, right-click your own name and open **Permissions** > **Client Permissions**.
2. Assign yourself explicit `75` power for **Group Member Add Power**, **Group Member Remove Power**, and **Group Modify Power**.
   - *Security result:* If you ever remove your own *Server Admin* group by accident, you still have enough client-level authority to restore it.

![Emergency Owner Lockout Prevention](/assets/images/ts/EmergencyOwnerLockoutPrevention.png)

---

## 3. Pre-Configure Channel Group Permissions

1. Go to **Permissions** > **Channel Groups**.
2. Select each real, non-default channel group (for example *Voice*, *Operator*, *Channel Admin*), excluding the default *Guest* group.
3. Under **Channel** > **Access**, enable all three join permissions:
   - `b_channel_join_permanent` (Join Permanent Channels)
   - `b_channel_join_semi_permanent` (Join Semi-Permanent Channels)
   - `b_channel_join_temporary` (Join Temporary Channels)
4. Set **Channel Join Power** = `50` and **Channel Subscribe Power** = `50` for assigned groups (for example *Voice*).

![Emergency Owner Lockout Prevention](/assets/images/ts/ChannelGroupPermissions.png)

---

## 4. Lock Down Channels & Hide Occupants (Zero-Power Channel Protection)

To secure a channel so unauthorized users cannot enter or see who is inside:

### Step 4.1: Revoke Channel Join & Subscribe Power

1. Right-click the target channel and select **Channel Permissions**.
2. Under **Channel** > **Access**:
   - Disable **Join Permanent**, **Join Semi-Permanent**, and **Join Temporary**.
   - Set **Channel Join Power** to `0`.
   - Set **Channel Subscribe Power** to `0`.
   - *Result:* Channel-level zero power overrides server-group access and makes the channel effectively invisible to unauthorized users.

![Emergency Owner Lockout Prevention](/assets/images/ts/user_channel.png)

### Step 4.2: Set High Required Channel Powers

1. Right-click the channel and open **Edit Channel** or adjust the channel permissions:
   - **Needed Channel Join Power**: `50` (or `75` for owner-only channels)
   - **Needed Channel Subscribe Power**: `50` (or `75` for owner-only channels)
   - **Needed Channel Description View Power**: `50` (or `75` for owner-only channels)
   - **Needed Channel Modify Power**: `50` (or `75` for owner-only channels)
   - **Needed Channel Delete Power**: `70` (or `75` for owner-only channels)
2. Save the changes.
   - *Result:* Outside users see the channel as empty, and occupant lists and movement notifications remain hidden.

---

## 5. Grant Access to Specific Members

To give a single user access to a locked channel:

1. Drag the user into the secure channel.
2. Right-click the user and choose **Set Channel Group**, then assign a non-guest group such as **Voice**.
3. *Hierarchy rule:* **Channel Group Permissions > Channel Permissions > Server Group Permissions**.
   - Because the channel group has `50` join and subscribe power, it overrides the channel's zero-power lock and satisfies the channel requirement.

---

## 6. Operational Server Settings & Privacy

1. **Lock Down Guest Whispers**:
   - Under **Permissions** > **Server Groups** > **Guest**, set **Client Whisper Power** = `-100`.
   - *Result:* Prevents spammers and new users from broadcasting whisper audio across the server.
2. **Reserved Slots**:
   - Right-click the server and open **Edit Virtual Server**.
   - Set **Reserved Slots** (for example `3`).
   - Enable `b_client_use_reserved_slot` on admin groups so staff can still connect when the server is full.
3. **Host Message Protection**:
   - Use **Modal Log** or **None** in the virtual server settings.
   - Do not use **Modal Quit**; it can disconnect clients in a loop.
4. **Server List Privacy**:
   - Disable **Enable reporting to server list** if you do not want your server publicly listed.
5. **Audit Logging**:
   - Turn on server logging and monitor admin actions, permission changes, and channel modifications.

---

---

## 7. TeamSpeak Admin Setup

**Remove** is either right click and remove, or uncheck

##Global - Nothing Set

---

###Vitrtual Server

    Information - All Checked

    Administration

        - Remove \*\*View List of Availabe Privilage Keys\*\* \`b_virtualserver_token_list\`

        - Remove \*\*Create New Privilage Key\*\* \`b_virtualserver_token_add\`

        - Remove \*\*ServerQuerry: Write to Virtual Log\*\* \`b_virtualserver_log_add\`

    Settings

        - Remove \*\*Modify Virtual Server Name\*\* \`b_virtualserver_modify_name\`

        - Remove \*\*Modify Virtual Server Max Clients\*\* \`b_virtualserver_modify_maxclients\`

        - Remove \*\*Modify Virtual Server Reserved Slots\*\* \`b_virtualserver_modify_reserved_slots\`

        - Remove \*\*Modify Virtual Server Default Server Goup\*\* \`b_virtualserver_modify_default_servergroup\`

        - Remove \*\*Modify Virtual Server Default Channel Goup\*\* \`b_virtualserver_modify_default_channelgroup\`

        - Remove \*\*Modify Virtual Server Default Admin Goup\*\* \`b_virtualserver_modify_default_channeladmingroup\`

        - Remove \*\*Modify Virtual Server Force Silence Limit\*\* \`b_virtualserver_modify_channel_forced_silence\`

        - Remove \*\*Modify Virtual Server Complaint Settings\*\* \`b_virtualserver_modify_complain\`

        - Remove \*\*Modify Virtual Server AntiFlood Settings\*\* \`b_virtualserver_modify_antiflood\`

        - Remove \*\*Modify Virtual Server FileTransfer Settings\*\* \`b_virtualserver_modify_ft_settings\`

        - Remove \*\*Modify Virtual Server FileTransfer Quotas\*\* \`b_virtualserver_modify_ft_quotas\`

        - Remove \*\*Modify Virtual Server Host Message\*\* \`b_virtualserver_modify_hostmessage\`

        - Remove \*\*Modify Virtual Server Host Banner\*\* \`b_virtualserver_modify_hostbanner\`

        - Remove \*\*Modify Virtual Server Host Button\*\* \`b_virtualserver_modify_hostbutton\`

        - Remove \*\*Modify Virtual Server Port\*\* \`b_virtualserver_modify_port\`

        - Remove \*\*Modify Virtual Server Log Settings\*\* \`b_virtualserver_modify_log_settings\`

---

###Channel

    Information - All Checked

    Administration - All Default

    Modify

        - pow \*\*Channel Modify Power\*\* \`i_channel_modify_power\` set to \`70\`

    Delete

        - pow \*\*Channel Delete Power\*\* \`i_channel_delete_power\` set to \`70\`

    Access

        - remove \*\*Ignore Channel Passwords\*\* \`b_channel_join_ignore_password\`

        - pow \*\*Channel Join Power\*\* \`i_channel_join_power\` set to \`70\`

        - pow \*\*Channel Subscribe Power\*\* \`i_channel_subscribe_power\` set to \`70\`

        - pow \*\*Channel Description Power\*\* \`i_channel_description_view_power\` set to \`70\`

    Root Base Channel

        - pow \*\*Channel Permission Modify Power\*\* \`i_channel_permission_modify_power\` set to \`70\`

---

\*\*\*Group

    Information

        - Remove \*\*View List Of Client Permissions\*\* \`b_virtualserver_client_permission_list\`

    Create - All Checked

    Modify

        - pow \*\*Group Modify Power\*\* \`i_group_modify_power\` set to \`70\`

        - pow \*\*Needed Group Modify Power\*\* \`i_group_needed_modify_power\` \`75\`

        - pow \*\*Group Member Add Power\`i_group_member_add_power\` set to \`70\`

        - pow \*\*Needed Group Member Add Power\`i_group_member_add_power\` \`75\`

        - pow \*\*Group Member Remove Power\*\* \`i_group_member_remove_power\` set to \`70\`

        - pow \*\*Needed Group Member Remove Power\*\* \`i_group_needed_member_remove_power\` \`75\`

        - pow \*\*Permission Modify Power \`i_permission_modify_power\` set to \`70\`

    Root Base Group

        - \*\*Icon ID\*\* \`i_icon_id\` Set to the desired icon for the group

        - num \*\*Show Group Name in Tree\*\* \`i_group_show_name_in_tree\` set to 1

---

\*\*\*Client

    Information

        - All default

    Administration

        - pow \*\*Client Kick from Server Power\*\* \`i_client_kick_from_server_power\` set to 70

        - pow \*\*Client Kick from Channel Power\*\* \`i_client_kick_from_channel_power\` set to 70

        - pow \*\*Client Ban From Server Power\*\* \`i_client_ban_power\` set to 70

        - pow \*\*Client Move Power\*\* \`i_client_move_power\` set to 70

        - pow \*\*Client Complain Power\*\* \`i_client_complain_power\`

        - Remove \*\*Delete own Ban Rules\*\* \`b_client_ban_delete_own\`

        - Remove Delete own Ban Rules \`b_client_ban_delete\`

        - num Max time for Ban Rules in seconds \`i_client_ban_max_bantime\` set to time, -1 is permanent

    Basic

        - pow Private Textmessage Power \`i_client_private_textmessage_power\` set to 70

        - pow Client Talk Power\*\* \`i_client_talk_power\` set to 70

        - pow Client Poke Power\*\* \`i_client_poke_power\` set to 70

        - pow \*\*Needed Poke Power\*\* \`i_client_needed_poke_power\` set to 70

    Modify

        - Remoce \*\*Modify all Client Descriptions\*\* \`b_client_modify_description\`

        - Remove \*\*Create a ServerQuerry Account\*\* \`b_client_create_modify_serverquery_login\`

        - Remove \*\*Skip Channel Group & Channel Permissions\*\* \`b_client_skip_channelgroup_permissions\`

---

\*\*\*File FileTransfer

    - \*\*Need to research these permissions\*\*
   - Monitor administrative actions, permission edits, and connections via **Tools** > **Server Log**.
  

