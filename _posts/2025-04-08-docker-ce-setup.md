---
title: Docker CE setup
author: Dan
date: 2025-04-08 02:00:00 -400
categories: [Server, Documentation, Docker]
tags: [homelab,documentation,server,docker]
---

# Docker and Docker Compose Setup

## Uninstall Old Versions

Before installing the latest version of **Docker**, ensure that any previous versions have been uninstalled:

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

## Install Docker Repositories

Update your package list and add the **Docker** official **GPG key**:

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

## Install Docker and Docker Compose

Install the **Docker CE** package, **Docker Compose** plugin, and other requirements:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Add User to the Docker Group

To manage **Docker** as a **non-root** user, add your user to the `docker` group:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

**Note**: You must log off and back in for the group changes to take effect.

## Verify Installation

Run the **Hello World** Docker container without using `sudo`:

```bash
docker run hello-world
```

If it runs without errors, the installation was successful.

To list all installed containers, use:

```bash
docker ps -a
```

To stop and remove a container:

```bash
docker stop <container_id_or_name>
docker rm <container_id_or_name>
```
