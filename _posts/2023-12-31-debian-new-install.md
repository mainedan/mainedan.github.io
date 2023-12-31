---
title: Debian 12 New Install
author: Dan
date: 2023-12-30 00:00:00 -400
categories: [Debian, linux, Documentation]
tags: [homelab,documentation,debian,linux]
---

# Debian 12 post install

* libre office - swith to flatpak version
	 > sudo apt-get remove --purge libreoffice*
  > sudo apt-get clean
  > sudo apt-get autoremove

	* uninstall in gnome software
	* uninstall remaining fragments
		> sudo apt remove libreoffice-common/ libreoffice-core/
libreoffice-gnome/ 
libreoffice-gtk3/
libreoffice-help-en-us/ 
libreoffice-calibri/ libreoffice-style-elementary/

* Install libreoffice in gnome software flathub version

* To switch desktop envirements
 	> sudo taskel
*	select what DE to use
	reboot
*	select username then gear icon then select whick DE to use

* NVidia drivers
	gnome software / select the hamburger / repositories
	select non-free software
	close and select reload
	sudo apt update && sudo apt install nvidia-driver

Steam
	install steam in gnome software
	install steam devices to be able to use controlers
	sudo apt update && sudo apt install steam-devices

codecs
	sudo apt install libavcodec-extra vlc

backport repo
	sudo nano /etc/apt/sources.list.d/backport.list
	deb http://deb.debian.org/debian/bookworm-backports main
	sudo apt update
