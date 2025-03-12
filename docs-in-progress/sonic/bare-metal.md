---
description: 'Author: [ jLeopoldA ]'
---

# Bare Metal

## System Requirements

| CPU                  | OS                 | RAM                | DISK |
| -------------------- | ------------------ | ------------------ | ---- |
| Minimum: 4 Cores     | Ubuntu 24.04.2 LTS | Minimum: 32GB      | 1TB  |
| Recommended: 8 Cores | Ubuntu 24.04.2 LTS | Recommended: 128GB | 1TB  |

{% hint style="info" %}
The Sonic Archive Node has a size of 588GB as of 3/12/2025.
{% endhint %}

{% hint style="warning" %}
Sonic requires GO v1.22.0 or greater to run.
{% endhint %}

## Pre-Requisites

#### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
sudo apt install -y git gcc make --fix-missing
```

#### Install GO

{% hint style="danger" %}
Sonic requires GO v1.22.0 or greater to run.
{% endhint %}

```bash
# Remove previous installation of GO
rm -rf /usr/local/go # For GO installations locacated within /usr/local/go
rm -rf /usr/local/bin/go # For GO installations located within /usr/local/bin/go

# Download GO
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz

# Extract and place within /usr/local
tar -xzf go1.22.0.linux-amd64.tar.gz -C /usr/local && rm go1.22.0.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

### Firewall Configuration

#### Set Explicit UFW Rules

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow Connections for Sonic

```bash
sudo ufw allow 18545
sudo ufw allow 18546
sudo ufw allow 5050
```

#### Enable Firewall

```bash
sudo ufw enable
```

#### View Current UFW Rules / Status

```bash
sudo ufw status verbose
```

## Set up Sonic

### Create Directories

```bash
mkdir -p /var/lib/sonic/configuration
mkdir -p /var/lib/sonic/database
```

### Install Sonic

#### Download Sonic

```bash
git clone https://github.com/0xsoniclabs/Sonic.git
cd Sonic
git fetch --tags && git checkout -b v2.0.1 tags/v2.0.1
```

#### Build Sonic

```bash
make all # Build Sonic
sudo cp build/sonic* /usr/local/bin/ 
```

### Prime Sonic State DB

```bash
cd /var/lib/sonic/configuration

# Download Genesis File
wget https://genesis.soniclabs.com/sonic-mainnet/genesis/sonic.g

# Prime DB
sonictool --datadir /var/lib/sonic/database --cache 12000 \
genesis /var/lib/configuration/sonic.g
```

## Create System Service

```bash
echo "[Unit]
Description=Sonic Node
After=network.target
StartLimitIntervalSec=200
StartLimitBurst=5

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/Sonic
ExecStart=/usr/local/bin/sonicd \
	--http --http.addr=0.0.0.0 --http.port=18545 \
	--http.corsdomain=* --http.vhosts=* \
	--http.api=eth,web3,net,ftm,txpool,abft,dag,trace,debug \
	--ws --ws.addr=0.0.0.0 --ws.port=18546 --ws.origins=* \
	--ws.api=eth,web3,net,ftm,txpool,abft,dag,trace,debug \
	--datadir /var/lib/sonic/database --cache 12000 \
	--nat extip:<THE_IP_ADDRESS_OF_THIS_COMPUTER>
KillSignal=SIGTERM
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/sonic.service
```

## Run Sonic Archive Node

#### Start Sonic Node

```bash
systemctl enable sonic.service # Enable Sonic Service
systemctl start sonic.service # Start Sonic Service
systemctl status sonic.service # Check status of Sonic Service
```

### Query Sonic Node

#### Check Sync Status

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0","method":"eth_syncing", "params":[], "id":1}' \
http://localhost:18545

# Response will resemble the below when node is done syncing. 
# Sync time was 6 days for the Author of this guide.
{"jsonrpc":"2.0","id":1,"result":false}
```

#### Get Current Block Number of Node

```bash
curl -H "Content-Type: application/json" \
-X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber", "params":[], "id":1}' \
http://localhost:18545

# Response should resemble the below:
{"jsonrpc":"2.0","id":1,"result":"0xca8227"}
```

## References

{% embed url="https://docs.soniclabs.com/sonic/node-deployment/archive-node" %}



