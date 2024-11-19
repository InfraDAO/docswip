---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 💻 Erigon

## System Requirements

<table><thead><tr><th width="150" align="center">CPU</th><th align="center">OS</th><th width="192" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">8+ cores CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 16 GB RAM</td><td align="center">500GB (SSD or NVMe)</td></tr></tbody></table>

{% hint style="info" %}
_The Erigon XLayer archive node has a size of 36GB on August 9th, 2024_
{% endhint %}

## Setup production Erigon

{% hint style="success" %}
This guide covers the installation of `CDK-Erigon`, a fork of Erigon, optimized for syncing with the XLayer network.
{% endhint %}

{% hint style="danger" %}
CAUTION: During the Chain Integration Process, InfraDAO noticed some POI divergencies when using `CDK-Erigon`. Consider following the the [**ZKNODE**](zk-node.md) **(clickable)** guide instead.
{% endhint %}

## Pre-Requisites

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y libgtest-dev libomp-dev libgmp-dev git make wget aria2 gcc pkg-config libusb-1.0-0-dev libudev-dev jq g++ curl libssl-dev screen apache2-utils build-essential
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
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw enable
</strong></code></pre>

To check the status of UFW and see the current rules

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw status verbose
</strong></code></pre>

### Install GO

{% hint style="info" %}
Go version 1.20.7 is required to build cdk-rigon
{% endhint %}

```bash
sudo wget https://go.dev/dl/go1.20.7.linux-amd64.tar.gz && sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.20.7.linux-amd64.tar.gz && rm go1.20.7.linux-amd64.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin:/root/.local/bin' >> /root/.bashrc

source /root/.bashrc

#verify Go installation
go version
```

### Build Erigon RPC Node

{% hint style="warning" %}
Before you start, make sure that you have your own synced Ethereum L1 RPC URL ready.
{% endhint %}

#### Clone the Erigon repository and build cdk-erigon. Check the latest version at [releases](https://github.com/0xPolygonHermez/cdk-erigon/releases) page.

```bash
git clone https://github.com/0xPolygonHermez/cdk-erigon.git

cd cdk-erigon

git checkout v1.2.24 #[checkout the latest release version]

make cdk-erigon
```

#### Configure xLayer Mainnet Parameters

```bash
mkdir /root/data/erigon-data/xlayer-mainnet

mkdir /root/xlayer

cd /root/xlayer

sudo nano xlayerconfig-mainnet.yaml
```

#### Paste and modify parameters. Save by entering `ctrl+X` and `Y+ENTER`

```bash
datadir: /root/data/erigon-data/xlayer-mainnet
chain: xlayer-mainnet
http: true
private.api.addr: localhost:9091
zkevm.l2-chain-id: 196
zkevm.l2-sequencer-rpc-url: https://rpc.xlayer.tech
zkevm.l2-datastreamer-url: stream.xlayer.tech:8800
zkevm.l1-chain-id: 1
zkevm.l1-rpc-url: {L1 RPC URL}

zkevm.address-sequencer: "0xAF9d27ffe4d51eD54AC8eEc78f2785D7E11E5ab1"
zkevm.address-zkevm: "0x2B0ee28D4D51bC9aDde5E58E295873F61F4a0507"
zkevm.address-admin: "0x491619874b866c3cDB7C8553877da223525ead01"
zkevm.address-rollup: "0x5132A183E9F3CB7C848b0AAC5Ae0c4f0491B7aB2"
zkevm.address-ger-manager: "0x580bda1e7A0CFAe92Fa7F6c20A3794F169CE3CFb"

zkevm.l1-rollup-id: 3
zkevm.l1-first-block: 19218658
zkevm.l1-block-range: 2000
zkevm.l1-query-delay: 1000
zkevm.rpc-ratelimit: 250
zkevm.datastream-version: 3

externalcl: true
http.api: [eth, debug, net, trace, web3, erigon, zkevm]
http.addr: 0.0.0.0
http.port: 8545
```

{% hint style="info" %}
```bash
Replace {L1 RPC URL} with your synced endpoint
```
{% endhint %}

### **Launch Erigon Node**

#### Create systemd service for `cdk-erigon`

```bash
sudo nano /etc/systemd/system/cdk-erigon.service
```

Paste the configs and save by entering `ctrl+X` and `Y+ENTER`:

```bash
[Unit]
Description=cdk-erigon Service
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
WorkingDirectory=/root/cdk-erigon/build/bin/
ExecStart=/root/cdk-erigon/build/bin/cdk-erigon --config="/root/xlayer/xlayerconfig-mainnet.yaml"
KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target
```

#### Start cdk-erigon

<pre class="language-bash"><code class="lang-bash">sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl enable cdk-erigon.service #enable cdk-erigon service at system startup

sudo systemctl start cdk-erigon.service #start cdk-erigon
<strong>
</strong>sudo nano /etc/systemd/system/cdk-erigon.service #make changes in cdk-erigon.service file
</code></pre>

### Run _`curl`_ command in the terminal to check the status of your node

<pre class="language-bash"><code class="lang-bash"><strong>curl -H "Content-type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:8545
</strong></code></pre>

When it returns `false` then your node is fully synchronized with the network

### Monitor the logs for errors

```bash
sudo journalctl -fu cdk-erigon.service #follow logs of cdk-erigon.service
```

During the synchonization, you are expected to get following log messages from `cdk-erigon`:

<pre class="language-bash"><code class="lang-bash">[INFO] [08-04|10:39:44.828] [1/16 L1Syncer] Starting L1 sync stage
[INFO] [08-04|10:39:44.829] Starting L1 syncer thread
[INFO] [08-04|10:39:54.914] [1/16 L1Syncer] L1 Blocks processed progress (amounts): 254000/1235309 (20%)
[INFO] [08-04|10:40:04.915] [1/16 L1Syncer] L1 Blocks processed progress (amounts): 430000/1235309 (34%)
....
[INFO] [08-04|14:32:56.247] [13/16 LogIndex] Started
<strong>[INFO] [08-04|14:32:56.248] [13/16 LogIndex] processing              from=499948 to=3642707
</strong>[INFO] [08-04|14:33:37.995] [13/16 LogIndex] Finished
[INFO] [08-04|14:33:45.451] [p2p] GoodPeers
[INFO] [08-04|14:33:45.700] [txpool] stat                            pending=0 baseFee=0 queued=0 alloc=353.1MB sys=6.2GB
[INFO] [08-04|14:33:59.786] [14/16 TxLookup] Flushed buffer file     name=/root/data/erigon-data/xlayer-mainnet/temp/erigon-sortable-buf-1679142578
[INFO] [08-04|14:34:02.324] [14/16 TxLookup] Flushed buffer file     name=/root/data/erigon-data/xlayer-mainnet/temp/erigon-sortable-buf-2496799223
[INFO] [08-04|14:34:03.661] [15/16 DataStream] Starting...
[INFO] [08-04|14:34:03.662] [15/16 DataStream] no streamer provided, skipping stage
[INFO] [08-04|14:34:03.662] [16/16 Finish] Started
[INFO] [08-04|14:34:03.662] [16/16 Finish] Finished
</code></pre>

### References

{% embed url="https://github.com/okx/Deploy/blob/main/mainnet/setup-erigon-rpc.md" %}

{% embed url="https://www.okx.com/ru/xlayer/docs/developer/build-on-xlayer/quickstart" %}
