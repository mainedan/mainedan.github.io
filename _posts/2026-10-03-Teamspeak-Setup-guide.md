---
title:  
author: Mainedan
date: 2026-10-02 04:00:00 -400
categories: [Teamspeak, Documentation]
tags: [secureity,documentation,setup]
---




# TeamSpeak 3 Server Permissions & Security Master Guide

This comprehensive guide outlines the complete step-by-step procedure for configuring a TeamSpeak 3 server using the Advanced Permission System. It incorporates all specific permission configurations from the **TeamSpeak Admin Setup** blueprint to establish a secure administrative hierarchy, prevent server hijacking, lock down channels, hide channel occupants, configure granular channel access, and tune operational server settings.

---

## 1. Enable the Advanced Permission System

1. Open TeamSpeak 3.
2. Navigate to **Settings** > **Options**.
3. Under the **Application** tab, locate and check **Advanced permission system**.
4. Click **Apply** or **OK**.

---

## 2. Establish a Safe Administrative Hierarchy (Prevent Hijacking)

By default, the *Server Admin* group has absolute power (`75`). Giving other staff members this default group allows them to modify server settings, take away the owner's admin status, or hijack ownership.

### Step 2.1: Duplicate the Server Admin Group
1. Go to **Permissions** > **Server Groups**.
2. Right-click **Server Admin** and select **Copy**. Name the new group **Admin** (or *Sub-Admin*).
3. Assign distinct icons for visual identification (e.g., Red icon for the Main Owner/Server Admin group).

### Step 2.2: Comprehensive Admin Group Permission Configuration
Select the new **Admin** group and apply the precise permission removals and power caps below:

#### A. Group Modify & Power Caps (`Group` Tab)
- **Modify Settings**:
  - `i_group_modify_power` = `70`
  - `i_group_needed_modify_power` = `75` (ensures Server Admin group cannot be modified by Admins)
  - `i_group_member_add_power` = `70`
  - `i_group_needed_member_add_power` = `75`
  - `i_group_member_remove_power` = `70`
  - `i_group_needed_member_remove_power` = `75`
  - `i_permission_modify_power` = `70`
- **Information**:
  - **Remove** `b_virtualserver_client_permission_list` (**View List of Client Permissions**)
- **Root Base Group**:
  - `i_icon_id` = Set to desired group icon
  - `i_group_show_name_in_tree` = `1`

#### B. Channel Modify & Power Caps (`Channel` Tab)
- **Modify & Delete**:
  - `i_channel_modify_power` = `70`
  - `i_channel_delete_power` = `70`
  - `i_channel_permission_modify_power` (Root Base Channel) = `70`
- **Access**:
  - **Remove** `b_channel_join_ignore_password` (**Ignore Channel Passwords**)
  - `i_channel_join_power` = `70`
  - `i_channel_subscribe_power` = `70`
  - `i_channel_description_view_power` = `70`

#### C. Client Administration & Basic Powers (`Client` Tab)
- **Administration**:
  - `i_client_kick_from_server_power` = `70`
  - `i_client_kick_from_channel_power` = `70`
  - `i_client_ban_power` = `70`
  - `i_client_move_power` = `70`
  - `i_client_complain_power` = `70`
  - **Remove** `b_client_ban_delete` (**Delete Ban Rules**)
  - **Remove** `b_client_ban_delete_own` (**Delete Own Ban Rules**)
  - `i_client_ban_max_bantime` = Set max ban duration in seconds (e.g., `86400` for 24 hours; `-1` is permanent)
- **Basic**:
  - `i_client_private_textmessage_power` = `70`
  - `i_client_talk_power` = `70`
  - `i_client_poke_power` = `70`
  - `i_client_needed_poke_power` = `70`
- **Modify**:
  - **Remove** `b_client_modify_description` (**Modify All Client Descriptions**)
  - **Remove** `b_client_create_modify_serverquery_login` (**Create a ServerQuery Account**)
  - **Remove** `b_client_skip_channelgroup_permissions` (**Skip Channel Group & Channel Permissions**)

#### D. Virtual Server Settings & Token Protection (`Virtual Server` Tab)
- **Administration**:
  - **Remove** `b_virtualserver_token_add` (**Create New Privilege Key**)
  - **Remove** `b_virtualserver_token_list` (**View List of Available Privilege Keys**)
  - **Remove** `b_virtualserver_log_add` (**ServerQuery: Write to Virtual Log**)
- **Settings (Remove All To Prevent Global Server Tampering)**:
  - **Remove** `b_virtualserver_modify_name` (Modify Virtual Server Name)
  - **Remove** `b_virtualserver_modify_maxclients` (Modify Max Clients)
  - **Remove** `b_virtualserver_modify_reserved_slots` (Modify Reserved Slots)
  - **Remove** `b_virtualserver_modify_default_servergroup` (Modify Default Server Group)
  - **Remove** `b_virtualserver_modify_default_channelgroup` (Modify Default Channel Group)
  - **Remove** `b_virtualserver_modify_default_channeladmingroup` (Modify Default Admin Group)
  - **Remove** `b_virtualserver_modify_channel_forced_silence` (Modify Force Silence Limit)
  - **Remove** `b_virtualserver_modify_complain` (Modify Complaint Settings)
  - **Remove** `b_virtualserver_modify_antiflood` (Modify AntiFlood Settings)
  - **Remove** `b_virtualserver_modify_ft_settings` (Modify FileTransfer Settings)
  - **Remove** `b_virtualserver_modify_ft_quotas` (Modify FileTransfer Quotas)
  - **Remove** `b_virtualserver_modify_hostmessage` (Modify Host Message)
  - **Remove** `b_virtualserver_modify_hostbanner` (Modify Host Banner)
  - **Remove** `b_virtualserver_modify_hostbutton` (Modify Host Button)
  - **Remove** `b_virtualserver_modify_port` (Modify Server Port)
  - **Remove** `b_virtualserver_modify_log_settings` (Modify Log Settings)

#### E. Feature Grants (`Grant` Values)
To allow secondary admins to manage features without full power, set the **Grant** value to `70` for:
- **Priority Speaker** (`b_client_is_priority_speaker` Grant = `70`)
- **Icon ID** (`i_icon_id` Grant = `70`)
- **Group Sort ID** (`i_group_sort_id` Grant = `70`)

### Step 2.3: Emergency Owner Lockout Prevention
1. As the main Server Owner, right-click your own name > **Permissions** > **Client Permissions**.
2. Assign yourself `75` power explicitly for **Group Member Add Power**, **Group Member Remove Power**, and **Group Modify Power**.
   - *Security Result:* If you ever accidentally remove your own *Server Admin* group, you retain client-level authority to re-assign yourself back into the group.

---

## 3. Pre-Configure Channel Group Permissions

1. Go to **Permissions** > **Channel Groups**.
2. Select each real, non-default Channel Group (e.g., *Voice*, *Operator*, *Channel Admin* — excluding default *Guest*).
3. Under **Channel** > **Access**, enable all three join permissions:
   - `b_channel_join_permanent` (Join Permanent Channels)
   - `b_channel_join_semi_permanent` (Join Semi-Permanent Channels)
   - `b_channel_join_temporary` (Join Temporary Channels)
4. Set **Channel Join Power** = `50` and **Channel Subscribe Power** = `50` for assigned groups (e.g., *Voice*).

---

## 4. Lock Down Channels & Hide Occupants (Zeroing Channel Power)

To secure a channel so that unauthorized users cannot enter or see who is inside:

### Step 4.1: Forcefully Revoke Channel Join & Subscribe Power
1. Right-click the target channel and select **Channel Permissions**.
2. Under **Channel** > **Access**:
   - Enable `Join Permanent`, `Join Semi-Permanent`, and `Join Temporary`, then **untick/disable** them.
   - Set **Channel Join Power** to `0` and untick/disable it.
   - Set **Channel Subscribe Power** to `0` and untick/disable it.
   - *Result:* Channel-level zero power overrides any server group subscribe/join power held by a client, instantly vanishing channel contents from their view.

### Step 4.2: Set High Channel Power Requirements
1. Right-click the channel and select **Edit Channel** (or adjust in Channel Permissions):
   - **Needed Channel Join Power**: Set to `50` (or `75` for Owner-only channels).
   - **Needed Channel Subscribe Power**: Set to `50` (or `75` for Owner-only channels).
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

## 6. Configure Sub-Admin Universal Access (Skip Flags)

To allow trusted sub-admins to view and join all hidden channels without giving them full `b_client_skip_channelgroup_permissions`:
1. Go to **Permissions** > **Server Groups** and select **Sub-Admin**.
2. Under **Channel**:
   - Set **Needed Channel Join Power** = `0` and check **Skip**.
   - Set **Needed Channel Subscribe Power** = `0` and check **Skip**.
   - Set **Needed Channel Description Power** = `0` and check **Skip**.
3. Under **Channel** > **Access**:
   - Enable `Join Permanent`, `Join Semi-Permanent`, and `Join Temporary`, and check **Skip** for each.
   - *Result:* The **Skip** flag forces every channel to evaluate as having a requirement of `0` for this sub-admin group, bypassing channel restrictions while keeping normal members locked out.

---

## 7. Operational Server Settings & Privacy

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
5. **Audit Logging**:
   - Monitor administrative actions, permission edits, and connections via **Tools** > **Server Log**.


