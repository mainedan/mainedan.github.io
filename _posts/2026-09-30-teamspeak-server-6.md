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

### Step 2.1: Duplicate the Server Admin Group (currenty can only do this in ts3 client)

1. Go to **Permissions** > **Server Groups**.
2. Right-click **Server Admin** and select **Copy**. Name the new group **Admin** (or *Sub-Admin*).
3. Assign distinct icons for visual identification (e.g., Red icon for the Main Owner/Server Admin group).

![Duplicate the Server Admin Group](/assets/images/ts/server-admin-copy.png)

### Step 2.2: Restrict Secondary Admin Power Levels

Select the new **Admin** group and reduce its key modify and access permissions from `75` down to **`70`**:

   1. **Group Permissions** (`Group` > `Modify`):
      - Set **Group Modify Power** = `70`
      - Set **Group Member Add Power** = `70`
      - Set **Group Member Remove Power** = `70`
      - Set **Permission Modify Power** = `70`
   2. **Channel Permissions** (`Channel` > `Modify` & `Access` & `Delete`):
      - Set **Channel Modify Power** = `70`
      - Set **Channel Delete Power** = `70`
      - Set **Channel Permission Modify Power** = `70`
      - Set **Channel Join Power** = `70`
      - *Security Result:* Secondary admins cannot modify or delete owner-protected channels/groups (set at `75`), nor can they promote themselves or others to full *Server Admin*.

![ChannelPermissions](/assets/images/ts/ChannelPermissions.png)

![Restrict Secondary Admin Power Levels](/assets/images/ts/Restrict Secondary Admin Power Levels.png)

   3. **Virtual Server** (`Virtual Server` > `Settings`) Uncheck or remove permission:
      - Untick/disable **Modify Virtual Server Max Clients** (`b_virtualserver_modify_maxclients`).
      - Untick/disable **Modify Virtual Server Name** (`b_virtualserver_modify_name`).
      - Untick/disable **Modify Virtual Server Reserved Slots** (`b_virtualserver_modify_reserved_slots`).
      - Untick/disable **Modify Virtual Server `Default Server Group` `Default Channel Group` `Default Admin Group` `Force Silence Limit` `Complaint Settings` `AnitFlood Settings` `File Transfer Settings` `File Transfer Quotas` `Host Message` `Host Banner` `Host Button` `Virtual Server Port` `Server Log Settings`

![Virtual Server](/assets/images/ts/VirtualServer.png)

   4. **Clent** (`Client` > `Administration`)
      - Set **Client Kick From Server Power** = `70`
      - Set **Needed Client Kick From Server Power** = `75`
      - Set **Client Kick From Channel Power** = `70`
      - Set **Needed Client Kick From Channel Power** = `75`
      - Set **Client Ban From Server Power** = `70`
      - Set **Needed Client Ban From Server Power** = `75`
      - Set **Client Move Power** = `70`
      - Set **Needed Client Move Power** = `75`
      - Set **Client Complain Power** = `70`
      - Set **Needed Client Complain Power** = `75`
      - Remove Permission**Delete Own Ban Rules**
      - Remove Permission **Delete All Ban Rules**

   5. **Clent** (`Client` > `Basic`)
      - Set **Private Textmessage Power** = `70`
      - Set **Client Talk Power** = `70`
      - Set **Client Poke Power** = `70`
      - Set **Needed Poke Power** = `70` (set this to the other groups so guests can not poke anyone, or set to 75 so only the server owner/admin can poke)                  
      - Remove Permission**Client Wisper Power**
      - Remove Permission **Needed Client Permission Power**

   6. **Clent** (`Client` > `Modify`)
      - Set **Client Permission Modify Power** = `70`
      - Set **Needed Client Permission Modify Power** = `75`
      - Remove Permission**Skip Client Group & Channel Permission**

   7. **Clent** (`Client`)
      - Remove Permission***Client Permission Modify Power**

### Step 2.3: Revoke Dangerous Bypass Flags

1. Under **Client**:
   - Untick/disable **Skip Channel Group & Channel Permissions** (`b_client_skip_channelgroup_permissions`).
2. Under **Channel** > **Access**:
   - Untick/disable **Ignore Channel Passwords** (`b_channel_join_ignore_password`).
   - *Security Result:* Secondary admins are forced to respect channel passwords and cannot barge into secure private channels.

### Step 2.4: Block Backdoor Access & Privilege Key Exploits

1. Under **Virtual Server Administration**:
   - Untick/disable **Create New Privilege Key** (`b_virtualserver_token_add`).
   - Untick/disable **View List of Available Privilege Keys** (`b_virtualserver_token_list`).
2. Under **Group Information**:
   - Untick/disable **View List of Client Permissions** (`b_client_permission_list`).
   - Ensure individual client permission editing (`b_client_permission_modify`) is restricted/disabled.
   - *Security Result:* Admins cannot generate secret admin tokens or grant hidden permissions directly to individual accounts.

### Step 2.5: Configure Moderation Powers & Feature Grants (Client)

1. Set **Kick Power**, **Ban Power**, **Poke Power**, **Talk Power**, and **Private Message Power** to `70`.
   -
3. To allow secondary admins to manage features without full power, set the **Grant** value to `70` for:
   - **Priority Speaker** (`b_client_is_priority_speaker` Grant = `70`)
   - **Icon ID** (`i_icon_id` Grant = `70`)
   - **Group Sort ID** (`i_group_sort_id` Grant = `70`)
4. Under **Max Ban Time in Seconds**, enter a numeric limit (e.g., `86400` for 24 hours) to prevent secondary admins from issuing permanent bans.

### Step 2.6: Emergency Owner Lockout Prevention

1. As the main Server Owner, right-click your own name > **Permissions** > **Client Permissions**.
2. Assign yourself `75` power explicitly for **Group Member Add Power**, **Group Member Remove Power**, and **Group Modify Power**.
   - *Security Result:* If you ever accidentally remove your own *Server Admin* group, you retain client-level authority to re-assign yourself back into the group.

![Emergency Owner Lockout Prevention](/assets/images/ts/EmergencyOwnerLockoutPrevention.png)

---

## 3. Pre-Configure Channel Group Permissions

1. Go to **Permissions** > **Channel Groups**.
2. Select each real, non-default Channel Group (e.g., *Voice*, *Operator*, *Channel Admin* — excluding default *Guest*).
3. Under **Channel** > **Access**, enable all three join permissions:
   - `b_channel_join_permanent` (Join Permanent Channels)
   - `b_channel_join_semi_permanent` (Join Semi-Permanent Channels)
   - `b_channel_join_temporary` (Join Temporary Channels)
4. Set **Channel Join Power** = `50` and **Channel Subscribe Power** = `50` for assigned groups (e.g., *Voice*).

![Emergency Owner Lockout Prevention](/assets/images/ts/ChannelGroupPermissions.png)

---

## 4. Lock Down Channels & Hide Occupants (Zeroing Channel Power)

To secure a channel so that unauthorized users cannot enter or see who is inside:

### Step 4.1: Forcefully Revoke Channel Join & Subscribe Power (Teampeak 3 Client)

1. Right-click the target channel and select **Channel Permissions**.
2. Under **Channel** > **Access**:
   - Disable `Join Permanent`, `Join Semi-Permanent`, and `Join Temporary`, then **untick/disable** them.
   - Set **Channel Join Power** to `0` and untick/disable it.
   - Set **Channel Subscribe Power** to `0` and untick/disable it.
   - *Result:* Channel-level zero power overrides any server group subscribe/join power held by a client, instantly vanishing channel contents from their view.

![Emergency Owner Lockout Prevention](/assets/images/ts/user_channel.png)

### Step 4.2: Set High Channel Power Requirements

1. Right-click the channel and select **Edit Channel** (or adjust in Channel Permissions):
   - **Needed Channel Join Power**: Set to `50` (or `75` for Owner-only channels).
   - **Needed Channel Subscribe Power**: Set to `50` (or `75` for Owner-only channels).
   - **Needed Channel Description View Power**: set to `50` (or `75` for Owner-only channels).
   - **Needed Channel Modify Power**: set to `50` (or `75` for Owner-only channels).
   - **Needed Channel Delete Power**: set to `70` (or `75` for Owner-only channels).
2. Save changes.
   - *Result:* Outside users see the channel as completely empty. Occupant lists and movement notifications remain invisible.

---

## 5. Grant Access to Specific Members

To give a specific user entry and visibility into a locked channel:

1. Drag the user into the secure channel.
2. Right-click the user, navigate to **Set Channel Group**, and assign a non-guest group (e.g., **Voice**).
3. *Hierarchy Rule:* **Channel Group Permissions > Channel Permissions > Server Group Permissions**.
   - Because the Channel Group has `50` Join and Subscribe power, it overrides the channel's zero power and meets the channel's requirement of `50`.

---

## 6. Operational Server Settings & Privacy

1. **Lock Down Guest Whispers**:
   - Under **Permissions** > **Server Groups** > **Guest**, set **Client Whisper Power** = `-100`.
   - *Result:* Prevents spammers and new users from broadcasting whisper audio server-wide.
2. **Reserved Slots**:
   - Right-click server > **Edit Virtual Server** > set **Reserved Slots** (e.g., `3`).
   - Enable `b_client_use_reserved_slot` on admin groups so staff can connect when full.
3. **Host Message Protection**:
   - Under Virtual Server settings, use **Modal Log** or **None**. **Never use Modal Quit** (it disconnects clients in an infinite loop).
4. **Server List Privacy**:
   - Uncheck **Enable reporting to server list** if you do not want your server publicly listed.
5. **Audit Logging**:#New Admin Group

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
  

