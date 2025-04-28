---
description: 'Author: [ jLeopoldA ]'
---

# Bare Metal

## System Requirements

| CPU    | OS                 | RAM     | DISK  |
| ------ | ------------------ | ------- | ----- |
| 4 Core | Ubuntu 24.04.1 LTS | 8GB RAM | 500GB |

{% hint style="info" %}
Starknet Juno has a size of 337GB as of 4/28/2025.
{% endhint %}

## Pre-Requisites

{% hint style="info" %}
Starknet Juno requires Rust and GO.
{% endhint %}

{% hint style="warning" %}
## Before you start, make sure that you have your own synced Ethereum mainnet L1 RPC URL ready with WS port enabled
{% endhint %}

#### Update System

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
sudo apt install -y git gcc make --fix-missing

sudo apt-get install -y libjemalloc-dev libjemalloc2 pkg-config libbz2-dev
make install-deps
```

#### Install Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

#### Install GO

```bash
wget https://go.dev/dl/go1.24.0.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.24.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
source ~/.bashrc
go version
```

## Firewall Configuration

#### Set Explicit Firewall Rules

```bash
sudo ufw default deny incoming && sudo ufw default allow outgoing
```

#### Allow SSH

```bash
sudo ufw allow 22/tcp
```

#### Allow Connections for Juno

```bash
sudo ufw allow 6060 && sudo ufw allow 6061
```

#### Enable Firewall

```bash
sudo ufw enable
```

#### View Current UFW Rules and Status

```bash
sudo ufw status verbose
```

## Set up Starknet Juno

### Create Directories

```bash
# This is for the data directory
sudo mkdir -p /var/lib/juno/data
```

### Build Starknet Juno

#### Download Juno

```bash
cd /root

git clone https://github.com/NethermindEth/juno

cd juno
```

#### Build Juno

```bash
make juno
```

### Create System Service for Starknet Juno

```bash
echo "[Unit]
Description=Juno
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
StandardOutput=journal
StandardError=journal
WorkingDirectory=/root/juno
ExecStart=/root/juno/build/juno \
	--db-path /var/lib/juno/data \
	--eth-node wss://{Ethereum_L1_RPC_HERE} \
	--http --http-port 6060 --http-host 0.0.0.0 \
	--ws --ws-port 6061 --ws-host 0.0.0.0 \

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/juno.service
```

## Run Starknet Juno&#x20;

#### Start System Service

```bash
systemctl daemon-reload
systemctl enable juno.service # Enable System Service for Starknet Juno
systemctl start juno.service # Start Starknet Juno

# Additional Commands to stop node, check node status or to restart node.
systemctl stop juno.service # Stop Starknet Juno
systemctl status juno.service # View Status
systemctl restart juno.service # Restart Starknet Juno
```

#### Access Starknet Juno Logs

```bash
journalctl -fu juno.service -o cat

# The response should resemble the below:
16:51:43.165 28/04/2025 +02:00	INFO	sync/sync.go:371	Stored Block	{"number": 1355405, "hash": "0x24c0...503a", "root": "0x7e5e...0b16"}
16:52:14.866 28/04/2025 +02:00	INFO	sync/sync.go:371	Stored Block	{"number": 1355406, "hash": "0x164e...d85b", "root": "0xe791...87e3"}
16:52:46.050 28/04/2025 +02:00	INFO	sync/sync.go:371	Stored Block	{"number": 1355407, "hash": "0x5b97...d55a", "root": "0x7e69...6da5"}
16:53:17.701 28/04/2025 +02:00	INFO	sync/sync.go:371	Stored Block	{"number": 1355408, "hash": "0x5420...5413", "root": "0x4fc0...bc63"}
16:53:45.869 28/04/2025 +02:00	INFO	sync/sync.go:371	Stored Block	{"number": 1355409, "hash": "0x3bc6...2f56", "root": "0x6efc...84ab"}
16:54:18.243 28/04/2025 +02:00	INFO	sync/sync.go:371	Stored Block	{"number": 1355410, "hash": "0x2982...1f39", "root": "0x1780...0a40"}
16:54:45.824 28/04/2025 +02:00	INFO	sync/sync.go:371	Stored Block	{"number": 1355411, "hash": "0x3f62...d34f", "root": "0x2d2e...9e99"}
16:55:17.282 28/04/2025 +02:00	INFO	sync/sync.go:371	Stored Block	{"number": 1355412, "hash": "0x21a0...cac1", "root": "0x5ba3...bd73"}
16:55:48.740 28/04/2025 +02:00	INFO	sync/sync.go:371	Stored Block	{"number": 1355413, "hash": "0x6792...f914", "root": "0x6cd3...95d7"}
16:56:20.350 28/04/2025 +02:00	INFO	sync/sync.go:371	Stored Block	{"number": 1355414, "hash": "0x6fd6...ed77", "root": "0x359e...ea61"}
```

## Query Starknet Juno

#### Check Sync Status

```bash
curl -H "Content-Type: application/json" -X POST --data \
'{"jsonrpc":"2.0", "method":"starknet_syncing", "params":[], "id":1}' \ 
http://localhost:6060

# When Starknet Juno finishes syncing - you should receive the following response:
{"jsonrpc":"2.0","result":false,"id":1}
```

#### Check Block Number

```bash
curl -H "Content-Type: application/json" -X POST --data \
'{"jsonrpc":"2.0", "method":"starknet_blockNumber", "params":[], "id":1}' \
http://localhost:6060

# The response should be similar to the following:
{"jsonrpc":"2.0","result":1355424,"id":1}
```

## References

{% embed url="https://github.com/NethermindEth/juno" %}
