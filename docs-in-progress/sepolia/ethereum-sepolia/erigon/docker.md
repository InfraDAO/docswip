---
description: 'Author: [ jLeopoldA ]'
---

# Docker

## System Requirements

| CPU     | OS                 | RAM  | DISK  |
| ------- | ------------------ | ---- | ----- |
| 8 Cores | Ubuntu 24.04.1 LTS | 32GB | 1.1TB |

{% hint style="info" %}
The Ethereum Sepolia Archive Node using Erigon has a size of 585GB as of 4/4/2025
{% endhint %}

## Pre-Requisites

{% hint style="info" %}
This method of setting up an Ethereum Sepolia Archive Node uses Erigon, Docker, and Docker Compose
{% endhint %}

### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
```

### Set Up Firewall

#### Set Explicit Default Firewall Rules

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow RPC Connections with Erigon Sepolia

```bash
sudo ufw allow 8545
```

#### Allow P2P Communication

```bash
sudo ufw allow 30303
```

#### Allow Prometheus Monitoring

```bash
sudo ufw allow 9090
```

#### Enable Firewall

```bash
sudo ufw enable
```

#### Check Status / Current Rules of UFW

```bash
sudo ufw status verbose
```

### Install Docker & Docker-Compose

#### Installation

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
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install Docker Packages including Docker Compose
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
 
# Verify Docker Engine Installation
sudo docker run hello-world
```

## Build Ethereum Sepolia Archive Node with Erigon

### Create Directory

```bash
mkdir -p /root/sepolia && cd /root/sepolia
```

### Create docker-compose.yml

```bash
echo "services:
  erigon:
    image: "erigontech/erigon:latest"
    container_name: erigon-sepolia
    ports:
      - "30303:30303"
      - "8545:8545"
      - "9090:9090"
    volumes:
      - erigon_data:/root/erigon/.local/share/erigon
    command:
      - "--chain=sepolia"
      - "--http"
      - "--http.api=net,web3,eth,debug"
      - "--http.corsdomain=*"
      - "--http.addr=0.0.0.0"
      - "--http.port=8545"
      - "--http.vhosts=*"
    restart: unless-stopped
volumes:
  erigon_data:" > docker-compose.yml
```

### Run Ethereum Sepolia Archive Node

```bash
 # Run the below code from within /root/sepolia
 docker compose up -d
```

{% hint style="info" %}
Using Erigon for Ethereum Sepolia requires the Archive Node to download snapshots. When it is done you can begin querying and interacting with your node. See the next section for information regarding checking sync status.
{% endhint %}

## Interact with Sepolia Archive Node

#### Check Logs of Erigon Sepolia

{% hint style="info" %}
Checking Logs will allow you to view sync status.
{% endhint %}

```bash
docker compose logs erigon-sepolia
```

#### Stop Archive Node

```bash
docker stop erigon-sepolia
```

## Query Sepolia Archive Node

### Check Block Number

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":1}' http://localhost:8545

# Response should have a different "result" value.
{"jsonrpc":"2.0","id":1,"result":"0x754ee0"}
```
