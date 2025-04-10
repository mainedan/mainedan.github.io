---
title: Docker CE setup
author: Dan
date: 2025-04-08 02:00:00 -400
categories: [Server, Documentation, Docker]
tags: [homelab,documentation,server,docker]
---

# Docker and Docker Compose Setup 




## Make sure and previous versions have been uninstalled 

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done 
```
 

## Then install docker repositories 

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

## Then install docker and docker-compose 

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 
```
 

## Add the user to the docker group Manage docker as a NON-ROOT user 
 
```bash
sudo groupadd docker 

sudo usermod -aG docker $USER 
```
 

Log off and back in, then try the Hello World docker without sudo command 

```bash
docker run hello-world 
``` 

 

If it runs with out errors, all set 

To list installed docker containers type 

```bash
docker ps -a 
``` 

 

to stop and remove a docker container  

```bash
docker stop (first few letters of the docker container or the container name) 

docker rm  (first few letters of the docker container or the container name) 
```
 