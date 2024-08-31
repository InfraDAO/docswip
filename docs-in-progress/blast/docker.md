---
description: 'Authors: [ JleopoldA ]'
---

# Docker

## System Requirements

| CPU    | OS             | RAM      | DISK |
| ------ | -------------- | -------- | ---- |
| 4 Core | Ubuntu 22.04.4 | 16GB RAM | 1TB  |

## Blast

{% hint style="info" %}
In order to run Blast you need to utilize an L1 node for Blast to interact with. Instructions to do so can be found here.\
[https://docs.prylabs.network/docs/install/install-with-script](https://docs.prylabs.network/docs/install/install-with-script)
{% endhint %}

#### Official Docs

[https://docs.blast.io/about-blast](https://docs.blast.io/about-blast)

### Pre-Requisites

Update, upgrade, clean the system, and apply firewall management (ufw).

```bash
# Update, upgrade and clean system
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
```

```bash
# Set explicit default UFW rules
sudo ufw default deny incoming && sudo ufw default allow outgoing

# Allow SSH, HTTP and HTTPS
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### Install Docker & Docker Compose

The following code will install Docker & Docker Compose, both are necessary requirements to run Blast.

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

# Install Docker Packages
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
 
 # Verify Docker Installation is Successful
sudo docker run hello-world

# Install Docker-Compose
sudo apt-get update
sudo apt-get install docker-compose-plugin

# Verify Docker-Compose installation by checking the version
docker compose version

# Expected output
Docker Compose version vN.N.N
```

### Create Blast Directory

&#x20;The command "mkdir blast" will create a directory named "blast" within your current working directory. The second command will alter your current working directory to the newly created "blast" directory.&#x20;

```bash
mkdir blast && cd blast
```

### Clone Blast-IO Deployment Repository

The Blast deployment repository contains the necessary Docker Compose Configurations. We will obtain the Deployment repository while within our "blast" directory. \
The commands "git clone git@github.com:blast-io/deployment.git"  or\
"git clone https://github.com/blast-io/deployment.git" will download the repository.\
The command "cd deployment" will change your current working directory to the deployment directory.

```bash
# Download Deployment Repository using SSH
git clone git@github.com:blast-io/deployment.git
cd deployment

# If you receive errors running "git clone git@github.com:blast-io/deployment.git"
# You can download it this way instead
git clone https://github.com/blast-io/deployment.git
cd deployment
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into your newly created .env file.

```bash
# NETWORK should be mainnet or sepolia
NETWORK={YOUR_NETWORK} # Set to either mainnet or sepolia
GETH_DATA_DIR=blast-get-data # The path where you want to store blockchain data
L1_RPC_URL={YOUR_RPC_ENDPOINT_URL} # L1 node RPC endpoint URL
L1_RPC_KIND={YOUR_RPC_KIND} # Type of RPC provider (alcehmy, infura, any, etc)
OP_NODE_L1_BEACON={YOUR_L1_BEACON_API} # Your L1 Beacon api endpoint
```

### Start Docker Containers

```bash
docker compose up -d
```

This pulls the latests version of the pre-built Docker images and starts the necessary containers.&#x20;

### View Docker Logs

```bash
docker logs blast-geth -f --tail 100
```

### References
