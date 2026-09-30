---
title: TeamSpeak 3 Server Permissions & Security Configuration
author: Mainedan
date: 2026-05-08 04:00:00 -400
categories: [Teamspeak, Documentation]
tags: [teamspeak,documentation,permissions]
---

--------------------------------------------------------------------------------

# TeamSpeak 3 Server Permissions & Security Master Guide

---

description: This guide outlines the step-by-step procedure for configuring a TeamSpeak 3 or 6 server using the Advanced Permission System. It covers establishing a secure administrative hierarchy to prevent server hijacking, locking down channels, hiding channel occupants, configuring granular channel access, and tuning operational server settings.

Used for the guide

{% include embed/youtube.html id='CDzk2KbYcVk?si=okNxQvH_N758oqnc' %}
📺 [Watch Video](https://youtu.be/CDzk2KbYcVk?si=okNxQvH_N758oqnc)

---

## 1. Enable the Advanced Permission System (Only required if using TeamSpeak 3 Client)

1. Open TeamSpeak 3.
2. Navigate to **Settings** > **Options**.
3. Under the **Application** tab, locate and check **Advanced permission system**.
4. Click **Apply** or **OK**.

image:

  path: /assets/images/ts/ts.png

  alt: Teamspeak

## 2. Establish a Safe Administrative Hierarchy (Prevent Hijacking)

By default, the *Server Admin* group has absolute power (`75`). Giving other staff members this default group allows them to modify server settings, take away the owner's admin status, or hijack ownership.

### Step 2.1: Duplicate the Server Admin Group
1. Go to **Permissions** > **Server Groups**.
2. Right-click **Server Admin** and select **Copy**. Name the new group **Admin** (or *Sub-Admin*).
3. Assign distinct icons for visual identification (e.g., Red icon for the Main Owner/Server Admin group).

### Step 2.2: Restrict Secondary Admin Power Levels
Select the new **Admin** group and reduce its key modify and access permissions from `75` down to **`70`**:
1. **Group Permissions** (`Group` > `Modify`):
   - Set **Group Modify Power** = `70`
   - Set **Group Member Add Power** = `70`
   - Set **Group Member Remove Power** = `70`
   - Set **Permission Modify Power** = `70`
2. **Channel Permissions** (`Channel` > `Modify` & `Access`):
   - Set **Channel Modify Power** = `70`
   - Set **Channel Delete Power** = `70`
   - Set **Channel Permission Modify Power** = `70`
   - Set **Channel Join Power** = `70`
   - Set **Needed Channel Join Power** = `70`
   - Set **Needed Channel Subscribe Power** = `70`
   - Set **Needed Channel Description Power** = `70`
   - *Security Result:* Secondary admins cannot modify or delete owner-protected channels/groups (set at `75`), nor can they promote themselves or others to full *Server Admin*.

### Step 2.3: Revoke Dangerous Bypass Flags
1. Under **Client**:
   - Untick/disable **Skip Channel Group Permissions** (`b_client_skip_channelgroup_permissions`).
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

### Step 2.5: Configure Moderation Powers & Feature Grants
1. Set **Kick Power**, **Ban Power**, **Poke Power**, **Talk Power**, and **Private Message Power** to `70`.
2. To allow secondary admins to manage features without full power, set the **Grant** value to `70` for:
   - **Priority Speaker** (`b_client_is_priority_speaker` Grant = `70`)
   - **Icon ID** (`i_icon_id` Grant = `70`)
   - **Group Sort ID** (`i_group_sort_id` Grant = `70`)
3. Under **Max Ban Time in Seconds**, enter a numeric limit (e.g., `86400` for 24 hours) to prevent secondary admins from issuing permanent bans.

### Step 2.6: Emergency Owner Lockout Prevention
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
