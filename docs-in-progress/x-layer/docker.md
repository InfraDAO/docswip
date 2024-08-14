---
description: 'Authors: [jLeopoldA]'
---

# 🐳 Docker

## System Requirements

| CPU        | OS                          | Ram          | Disk          |
| ---------- | --------------------------- | ------------ | ------------- |
| vCpu 4 min | Used: Debian / Ubuntu 22.04 | 16GB minimum | 13.5 TB (SSD) |

{% hint style="info" %}
ARCHIVAL SIZE HERE of xlayer here
{% endhint %}

## Required Installations

{% hint style="info" %}
Requirements to create an xLayer node that acts as an Archive node are Docker, Docker-Compose, Geth, Prysm and zkEVM.
{% endhint %}

### Update

```bash
sudo apt update -y
```

### Install Docker and Docker-Compose

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

### Install Geth

{% hint style="info" %}
Geth acts as a L1 node to utilize Ethereum mainnet for zkEVM mainnet
{% endhint %}

```bash
# Enable the launchpad repository
sudo add-apt-repository -y ppa:ethereum/ethereum

# Install the stable version of go-ethereum
sudo apt-get update
sudo apt-get install ethereum

# To upgrade an exisiting Geth installation, stop the node and run the following
sudo apt-get update
sudo apt-get install etthereum
sudo apt-get upgrade geth
```

## Set up Execution Client (Geth) and Beacon Client (Prysm)

{% hint style="warning" %}
Depending on the speed of your device, syncing between Geth and Prysm may take a few hours.
{% endhint %}

### Set up Prysm

```bash
# Set up Prysm
# Create a folder called "ethereum" on your SSD
# Within the "ethereum" folder - create two sub folders: "consensus" and "execution"

# Navigate to "consensus" directory and run the following: 
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh

# The HTTP connection between your beacon node and execution node needs to be
# authenticated using a JWT token
./prysm.sh beacon-chain generate-auth-secret

# Move your generated "jwt.hex" file to the "ethereum" directory
mv jwt.hex ../
```

### Set up Geth

```bash
# Move the "geth" executable into your "execution" directory
# To check where your "geth" executable is located
whereis geth

# The output may resemble the below
geth: /usr/bin/geth # note returned location

# Move geth from returned location to your current "execution" directory
sudo mv /usr/bin/geth [location of execution directory]

# Navigate to your "execution" directory and run "geth"
./geth --mainnet --http --http.api eth,net,engine,admin --http.port 8546 --authrpc.jwtsecret=../jwt.hex --ws --ws.addr 0.0.0.0 --ws.port 8546 --ws.api eth,net,web3
```

### Run a Beacon Node using Prysm

```bash
# Navigate to your "consensus" directory and run the following to start
# your beacon node that connects to your local execution node
./prysm.sh beacon-chain --execution-endpoint=http://localhost:8551 --mainnet --jwt-secret=../jwt.hex --checkpoint-sync-url=https://beaconstate.info --genesis-beacon-api-url=https://beaconstate.info
```

{% hint style="warning" %}
As noted above, syncing between Prysm and Geth may take hours.
{% endhint %}

## Set up a zkEVM Node

```bash
# Make a directory
mkdir ZKEVM && cd ZKEVM

# DOWNLOAD ARTIFACTS
ZKEVM_NET=mainnet && ZKEVM_DIR=zkevm && ZKEVM_CONFIG_DIR=zkevm/config

curl -L https://github.com/0xPolygonHermez/zkevm-node/releases/latest/download/$ZKEVM_NET.zip > $ZKEVM_NET.zip && unzip -o $ZKEVM_NET.zip -d $ZKEVM_DIR && rm $ZKEVM_NET.zip
mkdir -p $ZKEVM_CONFIG_DIR && cp $ZKEVM_DIR/$ZKEVM_NET/example.env $ZKEVM_CONFIG_DIR/.env

# Create data folders
mkdir data && cd data
mkdir pool-db && mkdir state-db

# EDIT THIS env file with the following information:
cd ..
nano ./config/.env

# Provide the follow values to the .env keys
ZKEVM_NODE_ETHERMAN_URL = "http://[server name]:8546" # we avoid using localhost here
ZKEVM_NODE_STATEDB_DATA_DIR = "../data/state-db"
ZKEVM_NODE_POOLDB_DATA_DIR = "../data/pool-db"

# Run your zkEVM node
docker compose --env-file $ZKEVM_CONFIG_DIR/.env -f $ZKEVM_DIR/$ZKEVM_NET/docker-compose.yml up -d

# Check that all components are running
docker compose --env-file $ZKEVM_CONFIG_DIR/.env -f $ZKEVM_DIR/$ZKEVM_NET/docker-compose.yml ps
# The following containers should be viewable
# zkevm-rpc
# zkevm-sync
# zkevm-state-db
# zkevm-pool-db
# zkevm-power
```

### Query zkEVM Node

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":83}' http://localhost:8545
```

## Access Logs

```bash
# TO VIEW LOGS
docker logs [container id OR container name]
```

##







