---
description: 'Author: [ jLeopoldA ]'
---

# Docker

## System Requirements

| CPU | OS | RAM | DISK |
| --- | -- | --- | ---- |
|     |    |     |      |

{% hint style="info" %}
The Arbitrum Sepolia Archive Node has a size of \<SIZE HERE> as of \<DATE>
{% endhint %}

### Pre-Requisites

{% hint style="info" %}
This method of setting up an Arbitrum Sepolia Archive Node uses Docker and Docker-Compose.
{% endhint %}

## Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
```

## Install Docker & Docker-Compose

#### Installation Steps

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

## Set Up Firewall

#### Set Explicit Default Firewall Rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

#### Allow SSH

```
sudo ufw allow 22/tcp
```

#### Allow RPC Connection with Arbitrum Sepolia&#x20;

```bash
sudo ufw allow 8547 && sudo ufw allow 8548
```

#### Allow P2P Connections

```bash
sudo ufw allow 9642
```

#### Enable Firewall

```bash
sudo ufw enable
```

#### Check Status / Current Rules of Firewall

```bash
sudo ufw status verbose
```

## Build Arbitrum Sepolia Archive Node

{% hint style="info" %}
The Arbitrum Sepolia Archive Node requires an archival snapshot to run. However - this is provided within the docker-compose.yml as the following parameter and value.\
\
\--init.latest=archive
{% endhint %}

#### Create Directory

```bash
mkdir -p /root/arbitrum
```

#### Create docker-compose.yml

```bash
# From within /root/arbitrum run the below.
echo "services:
  arbitrum:
    image: offchainlabs/nitro-node:v3.4.0-d896e9c
    container_name: arbitrum
    restart: unless-stopped
    volumes:
      -  /root/arbitrum:/root/arbitrum/.arbitrum
    ports:
      -  "8547:8547"
      -  "9642:9642"
      -  "8548:8548"
    command:
      -  --init.latest=archive
      -  --parent-chain.connection.url=https://rpc-sepolia.rockx.com
      -  --parent-chain.blob-client.beacon-url=https://lodestar-sepolia.chainsafe.io
      -  --chain.id=421614
      -  --http.api=net,web3,eth
      -  --http.corsdomain=*
      -  --http.addr=0.0.0.0
      -  --http.vhosts=*
" > docker-compose.yml
```

### Run Archive Node

```bash
# Run the following command form within /root/arbitrum
docker compose up -d
```

## Interact with Arbitrum Sepolia Archive Node

#### Check Logs

```bash
docker compose logs arbitrum
```

#### Stop Node

```bash
docker compose stop arbitrum
```

## Query Arbitrum Sepolia Archive Node

### Check Block Number

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":1}' http://localhost:8547

# Response should have a different "result" value.
{"jsonrpc":"2.0","id":1,"result":"0x754ee0"}
```

