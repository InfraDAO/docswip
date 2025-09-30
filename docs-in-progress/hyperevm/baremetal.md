---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 💻 Baremetal

## System Requirements

<table><thead><tr><th width="157" align="center">CPU</th><th align="center">OS</th><th width="166" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">16+ cores CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 64 GB</td><td align="center">500GB+ (SSD or NVMe)</td></tr></tbody></table>

{% hint style="info" %}
_HyperEVM Mainnet archive node consists of Nanoreth (89Gb) and a hl-visor (265GB+) on September 30th, 2025_
{% endhint %}

## HyperEVM

{% hint style="success" %}
In this guide the installation process of nanoreth will be covered which is needed to sync an archival node able to serve historical requests on HyperEVM mainnet chain.&#x20;
{% endhint %}

## Pre-Requisites

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget aria2 gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-configv unzip
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

Allow remote RPC connections with the node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 3454
```

Allow remote P2P connections with Nanoreth and HL Nodes

```bash
sudo ufw allow 30303
sudo ufw allow 4002
sudo ufw allow 4001
sudo ufw allow 3999
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw enable
</strong></code></pre>

To check the status of UFW and see the current rules

```bash
sudo ufw status verbose
```

## Install dependencies

### Install Rust

<pre class="language-bash"><code class="lang-bash">sudo apt update &#x26;&#x26; sudo apt install -y clang llvm-dev libclang-dev
<strong>
</strong><strong>curl https://sh.rustup.rs -sSf | sh -s -- -y
</strong>
export PATH="$HOME/.cargo/bin:$PATH"
<strong>
</strong><strong>source ~/.bashrc
</strong></code></pre>

### Install AWS

{% hint style="info" %}
You will need to register an account on AWS and enable and configure Amazon S3 via [https://console.aws.amazon.com](https://console.aws.amazon.com)
{% endhint %}

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

aws configure

region = ap-northeast-1
output = json

aws_access_key_id = {Your AWS access key}
aws_secret_access_key = {Your AWS secret key}
```

## Install Nanoreth

```bash
git clone https://github.com/hl-archive-node/nanoreth.git

cd nanoreth

git checkout nb-20250915 (make sure you’ve pulled the latest build)

make install
```

### Create systemd service for nanoreth

```bash
sudo nano /etc/systemd/system/hl-nanoreth.service
```

```bash
[Unit]
Description=Hyperliquid nanoreth Service
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
WorkingDirectory=/root/nanoreth/
ExecStart=/root/.cargo/bin/reth-hl node \
  --block-source=/root/data/hl-nanoreth/evm-blocks \
#  --local-ingest-dir=/root/hl/data/evm_block_and_receipts/ \
#  --s3 \
  --ws \
  --port 30303 \
  --discovery.port 30303 \
  --ws.origins="*" \
  --ws.addr=0.0.0.0 \
  --http.port=3545 \
  --ws.port=3555 \
  --ws.api=ots,web3,debug,eth,txpool,net,trace \
  --http \
  --http.corsdomain="*" \
  --http.addr=0.0.0.0 \
  --http.api=ots,web3,debug,eth,txpool,net,trace \
  --forward-call \
  --rpc.gascap 650000000
KillSignal=SIGTERM
[Install]
WantedBy=multi-user.target
```

Save by entering `ctrl+X` and `Y+ENTER`

#### Before starting Nanoreth we want to obtain evm-blocks from block 0. There are two options for doing this:

* **Using a database shared by a community member** - _Fast but_ _not recommended_, as this relies on a third-party provider and archive availability is often unreliable

<pre class="language-bash"><code class="lang-bash">mkdir -p /root/data/hl-nanoreth/evm-blocks/ &#x26;&#x26; cd /root/data/hl-nanoreth/evm-blocks/
<strong>
</strong><strong>aria2c --file-allocation=none -c -x 10 -s 10 https://hl-archive-node.xyz/snapshot/evm-blocks.tar.zst
</strong></code></pre>

* **Obtaining the official archive from AWS.** This is the recommended method. Use the following command to download the blocks. Better do it in screen session as synchronizing is going to take up to 48 hrs

```bash

aws s3 sync s3://hl-mainnet-evm-blocks/ /root/data/hl-nanoreth/evm-blocks/ --request-payer requester
```

#### **Once the database has finished syncing**, start the NanoReth systemd service with the `--block-source=/root/data/hl-nanoreth/evm-blocks` flag enabled. This allows the node to load and process blocks from **genesis up to the most recent available block** before continuing live synchronization using either --s3 or local block sync (using Hl-visor)&#x20;

```bash
sudo systemctl daemon-reload

sudo systemctl enable hl-nanoreth.service

sudo systemctl start hl-nanoreth.service

sudo journalctl -u hl-nanoreth.service -f -n 100
```
