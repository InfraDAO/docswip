---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 🐳 Docker

## System Requirements

<table><thead><tr><th align="center">CPU</th><th align="center">OS</th><th width="254" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4-Core CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 8 GB RAM</td><td align="center"><p>150 GB+</p><p> (SSD or NVMe)</p></td></tr></tbody></table>

{% hint style="info" %}
_Firehose poller for Starknet has a size of 60GB on April 14th, 2025_
{% endhint %}

{% hint style="success" %}
The Firehose instrumentation service is added to a node for efficient capture and simple storage of blockchain data.&#x20;

Firehose extracts, transforms and saves blockchain data in a highly performant file-based strategy. Blockchain developers can then access data extracted by Firehose through binary data streams.

This guide shows how to set up the **Firehose** poller for Starknet with **Substreams** support. We use two components:

**`firehose-core (`**&#x72;eader, relayer, merger, firehos&#x65;**`)`**\
&#xNAN;**`firehose-starknet (`**&#x53;tarknet-specific fetche&#x72;**`)`**
{% endhint %}

{% hint style="warning" %}
## This method of setting up Firehose for Starknet assumes that you have a your own synced Starknet mainnet full (or archive) node (URL endpoint available) and Ethereum Mainnet L1 endpoint ready. In this guide, you'll be able to configure and run the entire Firehose stack with a single `docker run` command
{% endhint %}

### Pre-Requisites <a href="#pre-requisties" id="pre-requisties"></a>

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
​
sudo apt install -y wget curl screen git ufw
```

### Setting up Firewall <a href="#setting-up-firewall" id="setting-up-firewall"></a>

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### Enable Firewall

```bash
sudo ufw enable
```

## Install Docker

#### Run this command to remove any conflicting docker

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

#### Add Docker's official GPG key:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

#### Add the repository to ppt sources:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
```

#### Install docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test docker is working
sudo docker run hello-world

#Install docker compose

sudo apt-get update
sudo apt-get install docker-compose-plugin

# Test the docker version
docker compose version
```

## Setting up a domain name to access RPC

Get the IP address of the host machine, you can use the following command in a terminal or command prompt

```bash
curl ifconfig.me
```

Set an A record for a domain, you need to access the domain's DNS settings and create an A record that points to the IP address of the host machine. This configuration allows users to reach your domain by resolving the domain name to the specific IP address associated with your host machine.

{% embed url="https://youtu.be/QcNBLSSn8Vg" %}

**Create Starknet directory**

```bash
mkdir firehose && cd firehose
```

### Create Dockerfile.firehose-reader

```bash
sudo nano Dockerfile.firehose-reader
```

Paste and save:

```bash
FROM ghcr.io/streamingfast/firehose-core:v1.9.5 as core
FROM ghcr.io/streamingfast/firehose-starknet:6141dea as firestarknet

FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    ca-certificates curl vim jq tzdata \
    && rm -rf /var/lib/apt/lists/*

COPY --from=core /app/firecore /usr/local/bin/firecore
COPY --from=firestarknet /app/firestarknet /usr/local/bin/firestarknet

ENTRYPOINT ["firecore"]
```

Create .env file

