---
description: 'Authors: [jLeopoldA]'
---

# Docker

## System Requirements

<table><thead><tr><th width="235">CPU</th><th width="181">OS</th><th width="152">Ram</th><th>Disk</th></tr></thead><tbody><tr><td>Recommended: 4vCPU</td><td>Recommended: Ubuntu 18.04</td><td>Recommended: 16GB</td><td>2.5TB+ (SSD)</td></tr><tr><td>Min: 2vCPU</td><td>Used: Debian/Ubuntu 22.04</td><td>Min: 8GB</td><td></td></tr></tbody></table>

{% hint style="info" %}
The Fuse archival node has a size of 2.4TB on July 3rd, 2024
{% endhint %}

### Pre-Requisites

```sh
sudo apt update -y
```

### Install Docker

```sh
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
```

#### Install Docker Packages

```sh
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Verify Docker Engine Installation is Successful

```sh
sudo docker run hello-world
```

## Building a Node on Fuse with Nethermind client

{% hint style="success" %}
Since **08.2022** Fuse is moving from OE client to [Nethermind](https://nethermind.io/). To bootstrap Fuse archive node this guide covers the steps on how to build Nethermind from source and configure it to run for Fuse Network
{% endhint %}

### Clone Fuse Repository

```sh
$ git clone https://github.com/fuseio/fuse-network.git ~/Dev/fuse-network
```

### Download Quickstart

You can create an archival node using Quickstart.sh

```sh
# Enter the fuse-network folder
cd ~/Dev/fuse-network

# Download
wget -O quickstart.sh https://raw.githubusercontent.com/fuseio/fuse-network/master/nethermind/quickstart.sh

# Gain needed permissions
chmod 755 quickstart.sh

# Run Outline Example
./quickstart.sh -r [node_role] -n [network_name] -k [node_key]

# To create an archival node
./quickstart.sh -r explorer -n fuse -k [your_node_key_here]
```

```sh
# To ensure your archival node has been correctly started, run the below
curl -X POST -H "Content-type: application/json" --data \
'{"jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":1}' \
http://localhost:8545

# Your result should include a block number
```



{% hint style="success" %}
To access your archival node - use your current server's IP address with port 8545.\
Ex. http://localhost:8545
{% endhint %}

## Check Docker Logs

```sh
# Get Container ID
docker ps

# Desired Image name should be "fusenet/node:nethermind"
# Copy the Container ID and check your logs
docker logs [Container_ID]


```

