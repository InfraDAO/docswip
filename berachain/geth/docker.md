# 🐳 Docker

Authors: \[ Ankur | Dapplooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>8 vCPU</td><td>Ubuntu 22.04</td><td>16 GB</td><td> 2 TB</td></tr></tbody></table>

{% hint style="success" %}
_The Berachain node has a size of  1.6 TiB on  10, June, 2025._
{% endhint %}

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker
* curl
* jq

### **Commands**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io ufw -y jq -y
```
{% endcode %}

## Firewall Settings

### Check status & enable UFW&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw enable
</strong>sudo ufw status verbose
</code></pre>

### Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow SSH, HTTP, and HTTPS

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### Allow Remote connection

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 25445
```

## Setup Instructions

{% stepper %}
{% step %}
### Setup Project Directory

```bash
mkdir berachain ;
cd berachain ;
```

Create _setup-node.sh_ file.

```bash
#!/bin/bash

set -e

echo "Berachain Mainnet Archive Node Setup"

# Check prerequisites
if ! command -v docker >/dev/null 2>&1; then
    echo "ERROR: Docker is not installed"
    exit 1
fi

if ! command -v curl >/dev/null 2>&1; then
    echo "ERROR: curl is not installed"
    exit 1
fi

# Create directory structure
echo "Creating directories..."
rm -rf data config
mkdir -p data/geth data/beacond/config config

# Download configuration files
echo "Downloading configuration files..."
GENESIS_URL="https://raw.githubusercontent.com/berachain/beacon-kit/refs/heads/main/testing/networks/80094/genesis.json"
ETH_GENESIS_URL="https://raw.githubusercontent.com/berachain/beacon-kit/refs/heads/main/testing/networks/80094/eth-genesis.json"
KZG_TRUSTED_URL="https://raw.githubusercontent.com/berachain/beacon-kit/refs/heads/main/testing/networks/80094/kzg-trusted-setup.json"
CONFIG_TOML_URL="https://raw.githubusercontent.com/berachain/beacon-kit/refs/heads/main/testing/networks/80094/config.toml"
APP_TOML_URL="https://raw.githubusercontent.com/berachain/beacon-kit/refs/heads/main/testing/networks/80094/app.toml"
EL_PEERS_URL="https://raw.githubusercontent.com/berachain/beacon-kit/refs/heads/main/testing/networks/80094/el-peers.txt"

curl -s -o config/consensus-genesis.json "$GENESIS_URL" || { echo "ERROR: Failed to download consensus genesis"; exit 1; }
curl -s -o config/eth-genesis.json "$ETH_GENESIS_URL" || { echo "ERROR: Failed to download execution genesis"; exit 1; }
curl -s -o data/beacond/kzg-trusted-setup.json "$KZG_TRUSTED_URL" || { echo "ERROR: Failed to download KZG trusted setup"; exit 1; }
curl -s -o config/config.toml "$CONFIG_TOML_URL" || { echo "ERROR: Failed to download config.toml"; exit 1; }
curl -s -o config/app.toml "$APP_TOML_URL" || { echo "ERROR: Failed to download app.toml"; exit 1; }
curl -s -o config/el-peers.txt "$EL_PEERS_URL" || { echo "ERROR: Failed to download EL peers"; exit 1; }

# Generate JWT secret
echo "Generating JWT secret..."
if command -v openssl >/dev/null 2>&1; then
    openssl rand -hex 32 | tr -d '\n' | sed -e 's/^/0x/' > data/jwtsecret
elif command -v head >/dev/null 2>&1 && [ -f /dev/urandom ]; then
    head -c 32 /dev/urandom | xxd -p -c 32 | tr -d '\n' | sed -e 's/^/0x/' > data/jwtsecret
else
    echo "ERROR: Cannot generate JWT secret. Install openssl."
    exit 1
fi

# Get public IP
PUBLIC_IP=$(curl -s https://ipv4.icanhazip.com/ 2>/dev/null || curl -s https://api.ipify.org 2>/dev/null || echo "127.0.0.1")

# Configure Beacond
echo "Configuring Beacond..."
cp config/config.toml data/beacond/config/config.toml
cp config/app.toml data/beacond/config/app.toml

# Update config.toml with better P2P settings
sed -i 's/moniker = ".*"/moniker = "berachain-archive-node"/' data/beacond/config/config.toml
sed -i 's#laddr = "tcp://.*:26656"#laddr = "tcp://0.0.0.0:25456"#' data/beacond/config/config.toml
sed -i "s#external_address = \".*\"#external_address = \"$PUBLIC_IP:25456\"#" data/beacond/config/config.toml
sed -i 's#laddr = "tcp://.*:26657"#laddr = "tcp://0.0.0.0:26657"#' data/beacond/config/config.toml

# P2P connectivity settings
sed -i 's/max_num_inbound_peers = .*/max_num_inbound_peers = 40/' data/beacond/config/config.toml
sed -i 's/max_num_outbound_peers = .*/max_num_outbound_peers = 10/' data/beacond/config/config.toml
sed -i 's/dial_timeout = ".*"/dial_timeout = "10s"/' data/beacond/config/config.toml
sed -i 's/handshake_timeout = ".*"/handshake_timeout = "20s"/' data/beacond/config/config.toml
sed -i 's/pex = .*/pex = true/' data/beacond/config/config.toml
sed -i 's/seed_mode = .*/seed_mode = false/' data/beacond/config/config.toml
sed -i 's/allow_duplicate_ip = .*/allow_duplicate_ip = true/' data/beacond/config/config.toml

# Set better seed configuration - use official mainnet seeds
cat >> data/beacond/config/config.toml << 'EOF'

# Mainnet seeds for better connectivity
seeds = "9551cfb45e5220e28b397da58b022841cc4fd258@35.198.184.184:26656,da82fec973bb65b2dce04f5824ce27b5510adce1@35.198.137.87:26656,faeb10d59507e9e42a45fb0ed34bb1fa96c82d04@35.240.197.116:26656"
persistent_peers = ""
EOF

# Update app.toml
sed -i 's/pruning = ".*"/pruning = "nothing"/' data/beacond/config/app.toml
sed -i 's#rpc-dial-url = ".*"#rpc-dial-url = "http://bera-geth:8551"#' data/beacond/config/app.toml
sed -i 's#jwt-secret-path = ".*"#jwt-secret-path = "/jwtsecret"#' data/beacond/config/app.toml
sed -i 's#trusted-setup-path = ".*"#trusted-setup-path = "/beacond/kzg-trusted-setup.json"#' data/beacond/config/app.toml
sed -i 's/enabled = false/enabled = true/g' data/beacond/config/app.toml
sed -i 's#address = ".*:3500"#address = "0.0.0.0:3500"#' data/beacond/config/app.toml

cp config/consensus-genesis.json data/beacond/config/genesis.json

# Clean any existing Geth data to prevent state scheme conflicts
echo "Cleaning Geth data directory..."
rm -rf data/geth/*

# Initialize Geth
echo "Initializing Geth..."
docker run --rm \
    -v "$(pwd)/data/geth:/data" \
    -v "$(pwd)/config:/config" \
    ethereum/client-go:v1.15.10 \
    init --datadir=/data --db.engine=pebble --state.scheme=hash /config/eth-genesis.json

# Initialize Beacond
echo "Initializing Beacond..."
docker run --rm \
    -v "$(pwd)/data/beacond:/beacond" \
    ghcr.io/berachain/beacon-kit:v1.2.0 \
    init berachain --chain-id=mainnet-beacon-80094 --home=/beacond --overwrite

# Restore configurations
cp config/consensus-genesis.json data/beacond/config/genesis.json
cp config/config.toml data/beacond/config/config.toml
cp config/app.toml data/beacond/config/app.toml

# Re-apply configurations
sed -i 's/moniker = ".*"/moniker = "berachain-archive-node"/' data/beacond/config/config.toml
sed -i 's#laddr = "tcp://.*:26656"#laddr = "tcp://0.0.0.0:25456"#' data/beacond/config/config.toml
sed -i "s#external_address = \".*\"#external_address = \"$PUBLIC_IP:25456\"#" data/beacond/config/config.toml
sed -i 's#laddr = "tcp://.*:26657"#laddr = "tcp://0.0.0.0:26657"#' data/beacond/config/config.toml

# Re-apply P2P settings
sed -i 's/max_num_inbound_peers = .*/max_num_inbound_peers = 40/' data/beacond/config/config.toml
sed -i 's/max_num_outbound_peers = .*/max_num_outbound_peers = 10/' data/beacond/config/config.toml
sed -i 's/dial_timeout = ".*"/dial_timeout = "10s"/' data/beacond/config/config.toml
sed -i 's/handshake_timeout = ".*"/handshake_timeout = "20s"/' data/beacond/config/config.toml
sed -i 's/pex = .*/pex = true/' data/beacond/config/config.toml
sed -i 's/seed_mode = .*/seed_mode = false/' data/beacond/config/config.toml
sed -i 's/allow_duplicate_ip = .*/allow_duplicate_ip = true/' data/beacond/config/config.toml

# Add seeds again
cat >> data/beacond/config/config.toml << 'EOF'

# Mainnet seeds for better connectivity
seeds = "9551cfb45e5220e28b397da58b022841cc4fd258@35.198.184.184:26656,da82fec973bb65b2dce04f5824ce27b5510adce1@35.198.137.87:26656,faeb10d59507e9e42a45fb0ed34bb1fa96c82d04@35.240.197.116:26656"
persistent_peers = ""
EOF

sed -i 's/pruning = ".*"/pruning = "nothing"/' data/beacond/config/app.toml
sed -i 's#rpc-dial-url = ".*"#rpc-dial-url = "http://bera-geth:8551"#' data/beacond/config/app.toml
sed -i 's#jwt-secret-path = ".*"#jwt-secret-path = "/jwtsecret"#' data/beacond/config/app.toml
sed -i 's#trusted-setup-path = ".*"#trusted-setup-path = "/beacond/kzg-trusted-setup.json"#' data/beacond/config/app.toml
sed -i 's/enabled = false/enabled = true/g' data/beacond/config/app.toml
sed -i 's#address = ".*:3500"#address = "0.0.0.0:3500"#' data/beacond/config/app.toml

# Prepare bootnode string
EL_PEERS=$(cat config/el-peers.txt | tr '\n' ',' | sed 's/,$//')
echo "$EL_PEERS" > config/bootnode-string.txt

# Create environment file
cat > .env << EOF
PUBLIC_IP=$PUBLIC_IP
EL_BOOTNODES=$EL_PEERS
GETH_VERSION=v1.15.10
BEACOND_VERSION=v1.2.0
EOF

echo "Setup completed successfully"
echo "Start node: docker compose up -d"
```

make the scrip executable :

```bash
chmod +x setup-node.sh ;
```
{% endstep %}

{% step %}
**Create Docker Compose File**

Create _docker-compose.yml_ file.

```yaml
services:
  # Geth Execution Layer
  bera-geth:
    image: ethereum/client-go:${GETH_VERSION:-v1.15.10}
    container_name: bera-geth
    restart: unless-stopped
    stop_grace_period: 10m
    networks:
      - berachain
    ports:
      - "25403:25403/tcp" # P2P networking
      - "25403:25403/udp"
      - "25445:8545" # JSON-RPC
      - "25446:8546" # WebSocket
    volumes:
      - ./data/geth:/data
      - ./data/jwtsecret:/jwtsecret:ro
      - ./config/bootnode-string.txt:/bootnode-string.txt:ro
      - /etc/localtime:/etc/localtime:ro
    environment:
      - GETH_DATADIR=/data
    entrypoint: ["/bin/sh", "-c"]
    command:
      - |
        BOOTNODES=$$(cat /bootnode-string.txt) &&
        exec geth \
          --datadir=/data \
          --db.engine=pebble \
          --state.scheme=hash \
          --port=25403 \
          --discovery.port=25403 \
          --nat=extip:${PUBLIC_IP:-127.0.0.1} \
          --gcmode=archive \
          --syncmode=full \
          --networkid=80094 \
          --bootnodes="$$BOOTNODES" \
          --maxpeers=100 \
          --netrestrict="" \
          --cache=8192 \
          --http \
          --http.addr=0.0.0.0 \
          --http.port=8545 \
          --http.corsdomain=* \
          --http.api=eth,net,web3,txpool,debug,trace,admin,engine \
          --ws \
          --ws.addr=0.0.0.0 \
          --ws.port=8546 \
          --ws.origins=* \
          --ws.api=eth,net,web3,txpool,debug,trace,admin,engine \
          --authrpc.addr=0.0.0.0 \
          --authrpc.port=8551 \
          --authrpc.vhosts=* \
          --authrpc.jwtsecret=/jwtsecret

  # Beacond Consensus Layer
  bera-beacond:
    image: ghcr.io/berachain/beacon-kit:${BEACOND_VERSION:-v1.2.0}
    container_name: bera-beacond
    restart: unless-stopped
    stop_grace_period: 10m
    networks:
      - berachain
    ports:
      # P2P networking
      - "25456:25456/tcp"
      - "25456:25456/udp"
      - "25457:26657" # RPC
      - "25430:3500" # Beacon API
    volumes:
      - ./data/beacond:/beacond
      - ./data/jwtsecret:/jwtsecret:ro
      - /etc/localtime:/etc/localtime:ro
    environment:
      - BEACOND_HOME=/beacond
    command:
      - start
      - --home=/beacond

networks:
  berachain:
    driver: bridge
```
{% endstep %}

{% step %}
### &#x20;Execute Node Configuration

```bash
./setup-node.sh 

#Berachain Mainnet Archive Node Setup
#Creating directories...
#Downloading configuration files...
#Generating JWT secret...
#Configuring Beacond...
#Cleaning Geth data directory...
#Initializing Geth...
...
...
...
#Setup completed successfully
#Start node: docker compose up -d

```
{% endstep %}

{% step %}
### Start the Node

```bash
docker compose up -d 
```
{% endstep %}
{% endstepper %}

## Monitoring

### Monitor Logs of Docker Container&#x20;

```bash
docker ps 
docker logs bera-geth
docker logs bera-beacond
```

{% hint style="warning" %}
&#x20;Total time required to sync is \~4 days
{% endhint %}

## Sync Status

### Latest Block

```bash
 curl -X POST -H "Content-Type: application/json"   --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'   http://localhost:25445
```

_Response should look like:_

```json
{"jsonrpc":"2.0","id":1,"result":"0x5e7ac2"}
```

## REFERENCES

{% embed url="https://docs.berachain.com/nodes/quickstart" %}
Development Docs
{% endembed %}

{% embed url="https://berascan.com/" %}
Block Explorer
{% endembed %}
