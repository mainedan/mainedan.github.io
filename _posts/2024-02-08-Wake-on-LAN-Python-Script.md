---
title: Wake on Lan Python Script
author: Dan
date: 2024-02-08 02:00:00 -400
categories: [Website, Server, Documentation]
tags: [homelab,documentation,server]
---

# Wake-on-LAN Python Script

This document provides an overview of a Python script that uses the Wake-on-LAN protocol to wake up a computer over a network.

## Introduction

Wake-on-LAN (WoL) is an Ethernet or Token Ring computer networking standard that allows a computer to be turned on or awakened by a network message. The message is usually sent to the target computer by a program executed on a device connected to the same local area network, such as a smartphone.

## Python Script

Here's a Python script that implements the Wake-on-LAN protocol:

```python
import socket
import struct

def wake_on_lan(macaddress):
    """ Switches on remote computers using WOL. """

    # Check macaddress format and try to compensate.
    if len(macaddress) == 12:
        pass
    elif len(macaddress) == 12 + 5:
        sep = macaddress[2]
        macaddress = macaddress.replace(sep, '')
    else:
        raise ValueError('Incorrect MAC address format')

    # Pad the synchronization stream.
    data = ''.join(['FFFFFFFFFFFF', macaddress * 20])
    send_data = b''

    # Split up the hex values and pack.
    for i in range(0, len(data), 2):
        send_data = b''.join([send_data, struct.pack('B', int(data[i: i + 2], 16))])

    # Broadcast it to the LAN.
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)
    sock.sendto(send_data, ('<broadcast>', 7))

# Example use
wake_on_lan('01-23-45-67-89-AB')
```


# Usage
To use this script, replace '01-23-45-67-89-AB' with the MAC address of the device you want to wake up. Then, run the script on a device connected to the same network as the target computer.

# Requirements
The target computer must be configured to accept Wake-on-LAN packets. This setting is usually found in the BIOS or UEFI settings of the computer.

# Note
This script should be run in the same local network as the target computer. If you’re trying to wake a computer over the internet, you might need to set up port forwarding on your router. Always ensure you’re following good security practices when modifying network settings.

