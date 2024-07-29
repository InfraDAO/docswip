---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 💻 Baremetal

## System Requirements

|      CPU     |           OS           |      RAM     |         DISK        |
| :----------: | :--------------------: | :----------: | :-----------------: |
| 8+ cores CPU | Debian 12/Ubuntu 22.04 | => 16 GB RAM | 15GB+ (SSD or NVMe) |

{% hint style="info" %}
_The Boba Mainnet archive node has a size of 12GB on July 29, 2024_
{% endhint %}

## Boba

{% hint style="success" %}
Boba is built on the Optimistic Rollup developed by [Optimism](https://optimism.io/).

In this guide, we are walking through the process of setting up a Boba Mainnet archive node using a forked version of Optimism's op-erigon and op-node.&#x20;

Boba Network, a Layer 2 scaling solution for Ethereum, provides enhanced transaction throughput and reduced fees while maintaining Ethereum's security. By deploying a Boba archive node, you'll have access to the complete transaction history, enabling advanced queries and analytics
{% endhint %}

{% hint style="warning" %}
Before you start, make sure that you have your own synced Ethereum L1 RPC URL (e.g. Erigon) and L1 Consensus Layer Beacon endpoint with **`all historical blobs data`** (e.g. Lighthouse) ready.
{% endhint %}

## Pre-Requisites

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget aria2 gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config
```
{% endcode %}

### Setting up Firewall

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH

```bash
sudo ufw allow 22/tcp
```

Allow remote RPC connections with Blast Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 9545
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Allow P2P Connections

```bash
sudo ufw allow 30304/tcp
sudo ufw allow 30304/udp
```

Enable Firewall

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw enable
</strong></code></pre>

To check the status of UFW and see the current rules

```bash
sudo ufw status verbose
```

## Install dependencies

#### Required Software Dependencies

<table><thead><tr><th width="154">Dependency</th><th width="110" align="center">Version</th><th width="233">Version Check Command</th></tr></thead><tbody><tr><td><mark style="color:green;">go</mark></td><td align="center"><code>^1.21</code></td><td><code>go version</code></td></tr><tr><td><mark style="color:orange;">node</mark></td><td align="center"><code>^20</code></td><td><code>node --version</code></td></tr><tr><td><mark style="color:blue;">pnpm</mark></td><td align="center"><code>^8</code></td><td><code>pnpm --version</code></td></tr><tr><td><mark style="color:green;">foundry</mark></td><td align="center"><code>^0.2.0</code></td><td><code>forge --version</code></td></tr><tr><td><mark style="color:orange;">make</mark></td><td align="center"><code>^4</code></td><td><code>make --version</code></td></tr><tr><td><mark style="color:green;">yarn</mark></td><td align="center"><code>1.22.21</code></td><td><code>yarn --version</code></td></tr><tr><td><mark style="color:blue;">nvm</mark></td><td align="center"><code>0.39.3</code></td><td><code>nvm --verison</code></td></tr></tbody></table>

### Install GO

{% code overflow="wrap" fullWidth="false" %}
```bash
sudo wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz && sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz && rm go1.21.6.linux-amd64.tar.gz

#to verify Go installation
go version

#If it returns Command 'go' not found simply run 
echo 'export PATH=$PATH:/usr/local/go/bin:/root/.local/bin' >> /root/.bashrc

#and then apply changes with

source /root/.bashrc
```
{% endcode %}

### Install nvm

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

### Download foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
```

### Install foundry

```bash
foundryup

source /root/.bashrc
```

### Install node and yarn

```bash
nvm install 18.12.0 && npm install --global yarn && nvm use 18.12.0 && npm -g install pnpm

source /root/.bashrc
```

### Check if go and all dependancies are installed

```bash
go version
nvm -v
npm -v
yarn -v
pnpm -v
```

## Build the Rollup Node

Clone the Boba Monorepo

```bash
git clone https://github.com/bobanetwork/boba.git

cd boba
```

Check out the required release branch

Release branches are created when new versions of the `op-node` are created. Read through the [Releases page](https://github.com/bobanetwork/boba/tags) to determine the correct branch to check out.

```bash
git checkout v1.1.6
```

### Build op-node

```bash
make op-node
```

#### Create database directory and jwt secret file

```bash
mkdir /root/data/boba

openssl rand -hex 32 | tr -d "\n" > /root/data/boba/jwt.hex
```

### Create systemd service for op-node

```bash
sudo echo "[Unit]
Description=Boba op-node Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/boba/op-node/bin/
ExecStart=/root/boba/op-node/bin/op-node \
  --l1={L1 RPC endpoint} \
  --l1.beacon={L1 Beacon RPC endpoint} \
  --l2=http://0.0.0.0:8551 \
  --l2.jwt-secret=/root/data/boba/jwt.hex \
  --network=boba-mainnet \
  --plasma.enabled=false \
  --rpc.addr=0.0.0.0 \
  --rpc.port=8545

KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/op-node.service
```

{% hint style="warning" %}
```
Replace {L1 RPC endpoint}
```
{% endhint %}
