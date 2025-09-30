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

#### **Give Nanoreth some time to catch up and when it stops proceeding switch to --S3 sync by editing systemd service file:**

```bash
sudo systemctl stop hl-nanoreth.service

sudo nano /etc/systemd/system/hl-nanoreth.service
```

<pre class="language-bash"><code class="lang-bash"><strong>[Unit]
</strong>Description=Hyperliquid nanoreth Service
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
#  --block-source=/root/data/hl-nanoreth/evm-blocks \
#  --local-ingest-dir=/root/hl/data/evm_block_and_receipts/ \
  --s3 \
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
</code></pre>

```bash
sudo systemctl daemon-reload

sudo systemctl restart hl-nanoreth.service
```

With this option enabled, NanoReth will **fetch any missing blocks directly from AWS**, process them, and continue retrieving new data from there going forward.

From this point, you can choose to either **continue using `--s3-sync`** or **set up Hyperliquid’s `hl-visor` node** to fetch blocks locally, which provides **lower latency and no reliability on AWS**

## **Install HL-Visor**

Download the binary and grant the necessary permissions to make it executable

```bash
mkdir -p /root/hyperliquid/ && cd /root/hyperliquid/

curl https://binaries.hyperliquid.xyz/Mainnet/hl-visor > /root/hyperliquid/hl-visor && chmod a+x /root/hyperliquid/hl-visor
```

Create a `visor.json` which tells `hl-visor` which chain to connect to

```bash
echo '{"chain": "Mainnet"}' > /root/hyperliquid/visor.json
```

Verify the authenticity of the `hl-visor` binary

```bash
wget https://raw.githubusercontent.com/hyperliquid-dex/node/refs/heads/main/pub_key.asc

gpg --import /root/hyperliquid/pub_key.asc

gpg --verify hl-visor.asc hl-visor
```

### Add peers for syncing with the network

```bash
curl -X POST --header "Content-Type: application/json" --data '{ "type": "gossipRootIps" }' https://api.hyperliquid.xyz/info
```

The above command will fetch the list of live peers

**Create the gossip override configuration**

Use the following command to create `/root/hyperliquid/override_gossip_config.json`:

```bash
cat > /root/hyperliquid/override_gossip_config.json <<EOF
{
  "root_node_ips": [
    { "Ip": "<IP from list>" },
    { "Ip": "<IP from list>" },
    { "Ip": "<IP from list>" }
  ],
  "try_new_peers": false,
  "chain": "Mainnet"
}
EOF
```

* Replace `<IP from list>` with at least 5 additional IPs returned by the previous `curl` command

#### Create systemd file for hl-visor service:

```bash
sudo nano /etc/systemd/system/hl-visor.service
```

Paste:

```bash
[Unit]
Description=Hyperliquid Mainnet Service
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
LimitNOFILE=500000
WorkingDirectory=/root/hyperliquid/
ExecStart=/root/hyperliquid/hl-visor run-non-validator --replica-cmds-style recent-actions --serve-eth-rpc --disable-output-file-buffering
KillSignal=SIGHUP

[Install]
WantedBy=multi-user.target
```

#### Launch hl-visor

```bash
sudo systemctl daemon-reload

sudo systemctl restart hl-visor.service

sudo systemctl enable hl-visor.service

sudo systemctl status hl-visor.service

sudo systemctl disable hl-visor.service

sudo journalctl -fu hl-visor.service -n 100
```

### Troubleshooting

If you happen to run into following error:

```log
hl-visor[289715]: invalid ip address: 2a01:4f8:272:55a1::2
hl-visor[289715]: invalid ip address: 2a01:4f8:272:55a1::2
hl-visor[289715]: unable to query enough endpoints for public ip n_endpoints=1 ip_to_endpoints={"xx.xx.0.xxx": ["api.ipify.org"]}
```

Disable IPv6:

```bash
nano /etc/sysctl.conf
```

Scroll to the bottom and add the following lines

```bash
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
```

Apply the changes

```bash
sysctl -p
```

Once `hl-visor` has finished syncing, you can switch `Nanoreth` to use the blocks produced by `hl-visor` as its source:

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
  --local-ingest-dir=/root/hl/data/evm_block_and_receipts/ \
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

## Logs

`hl-nanoreth.service`:

<pre class="language-log"><code class="lang-log">2025-09-30T05:38:08.590719Z  INFO Forkchoice updated head_block_hash=0x6c24b5a41ed4749eeecb027927deca0f33a9fdbd1b69c55ac4665eec91e35ccc safe_block_hash=0x6c24b5a41ed4749eeecb027927deca0f33a9fdbd1b69c55ac4665eec91e35ccc finalized_block_hash=0x6c24b5a41ed4749eeecb027927deca0f33a9fdbd1b69c55ac4665eec91e35ccc
2025-09-30T05:38:09.479800Z  INFO Loading block data from "/root/hl/data/evm_block_and_receipts/hourly/20250930/5"
<strong>2025-09-30T05:38:09.553961Z  INFO Received block from consensus engine number=15207178 hash=0x2ccd43d458ee20aa22eb34757f8c7f463d19c2a574d7087bf2801706b3ff2349
</strong>2025-09-30T05:38:09.556894Z  INFO State root task finished state_root=0x0000000000000000000000000000000000000000000000000000000000000000 elapsed=3.637µs
2025-09-30T05:38:09.557031Z  INFO Block added to canonical chain number=15207178 hash=0x2ccd43d458ee20aa22eb34757f8c7f463d19c2a574d7087bf2801706b3ff2349 peers=1 txs=6 gas_used=1.16Mgas gas_throughput=402.41Mgas/second gas_limit=2.00Mgas full=57.9% base_fee=0.52Gwei blobs=0 excess_blobs=0 elapsed=2.877896ms
2025-09-30T05:38:09.557121Z  INFO Canonical chain committed number=15207178 hash=0x2ccd43d458ee20aa22eb34757f8c7f463d19c2a574d7087bf2801706b3ff2349 elapsed=51.146µs

</code></pre>

`hl-visor.service:`

```log
2025-09-30T05:41:48.253Z WARN >>> hl-node @@ serialized abci state for greeting @@ [abci_state.initial_height(): 744217000] @ [abci_state.height(): 747340000] @ [height: 747340000] @ [app_hash: AppHash(0x26c7a24d0478d5642840223902bcbe9214c623bf72f69aa3ecebde8483ea9922)] @ [serialized_abci_state.len(): 796340359]
2025-09-30T05:41:49.185Z WARN >>> hl-node @@ new app hashes reached quorum: [(747340000, QuorumAppHash { app_hash: AppHash(0x26c7a24d0478d5642840223902bcbe9214c623bf72f69aa3ecebde8483ea9922), hot_user_to_signature: Map({User(0x263294039413b96d25e4173a5f7599f8b3801504): Signature { r: 11226097415728839685190038623666713917190623830926729210141813323781111584450, s: 41611733720057584615483157272637821957617442682069963495194403651334751946595, v: 27 }, User(0xda6816df552c3f9e0fb64979fb357800d690d79b): Signature { r: 64996816189104273047422151237300995979151712004356590728032981323771342471738, s: 43857914805416652328827852965788456878785485757437346182906383839811362321239, v: 27 }, User(0xef2364db5db6f5539aa0bc111771a94ee47637fc): Signature { r: 16083230333297748674825901868221088981578128627302424425982620981396746852805, s: 52642217192111573387864443495565731691625857210130769552404787153265046789557, v: 28 }}) })]
2025-09-30T05:41:53.922Z WARN >>> hl-node @@ applied block 747340200
2025-09-30T05:42:01.989Z WARN >>> hl-node @@ applied block 747340300
```

## References

{% embed url="https://github.com/hyperliquid-dex/node" %}

{% embed url="https://github.com/hl-archive-node/nanoreth.git" %}

{% embed url="https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/nodes" %}
