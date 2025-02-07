---
description: 'Authors: Godwin'
---

# 💻 Baremetal

## System Requirements

|                 CPU                 |           OS           |    RAM    |     DISK    |
| :---------------------------------: | :--------------------: | :-------: | :---------: |
| 2 -4 Cores (Fastest per core speed) | Debian 12/Ubuntu 22.04 | 8 - 16 GB | 286GB (SSD) |

{% hint style="info" %}
_The gravity node has a size of 268GB on January 28, 2025_
{% endhint %}

## Offchain Labs ⛓️

{% hint style="info" %}
Official Docs [https://docs.arbitrum.io/node-running/how-tos/running-an-archive-node](https://docs.arbitrum.io/node-running/how-tos/running-an-archive-node)
{% endhint %}

### Pre-Requisites

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config
```

### Setting up Firewall

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH

```bash
sudo ufw allow 22/tcp
sudo ufw allow 8546
sudo ufw allow 8547
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

```bash
sudo ufw enable
```

### Install Docker

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

### Add Docker Official GPG Key

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Add the repository to ppt source

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
```

### Install Docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test docker is working
sudo docker run hello-world
```

#### Install go

Download the Go programming language distribution archive, extracts it to the "/usr/local" directory, and then removes the downloaded archive, effectively installing Go version 1.23.5 on the system.

```bash
wget https://go.dev/dl/go1.23.5.linux-amd64.tar.gz && \
rm -rf /usr/local/go && \
tar -C /usr/local -xzf go1.23.5.linux-amd64.tar.gz && \
rm go1.23.5.linux-amd64.tar.gz
```

Add /usr/local/go/bin to the `PATH` environment variable.

You can do this by adding the following line to your $HOME/.profile or /etc/profile (for a system-wide installation):

```bash
export PATH=$PATH:/usr/local/go/bin

 go version
```

### Build Nitro with Docker

```bash
git clone --recurse-submodules https://github.com/OffchainLabs/nitro/
cd nitro
docker build -t nitro .
```

### Copy Nitro binary from docker to `/root/nitro/build/bin`

```bash
docker cp $(docker run -it -d nitro):/usr/local/bin/nitro /root/nitro/build/bin/
/root/nitro/build/bin/nitro -h # test
```

### Create service to run Nitro Node

Create a local directory and replace `<some_local_dir>` with your path

```
mkdir -p /root/.gravity/share/nitro/datadir
```

&#x20;Create service

```
sudo nano /etc/systemd/system/gravity.service
```

```bash
[Unit]
Description=Gravity Alpha Mainnet Node
After=network.target

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
ExecStart=/root/nitro/build/bin/nitro \
  --parent-chain.connection.url=<ethereum_mainnet_rpc> \
  --persistent.chain=/root/.gravity/share/nitro/datadir/ \
  --persistent.global-config=/root/.gravity/share/nitro/ \
  --chain.id=1625 \
  --chain.name=conduit-orbit-deployer \
  --chain.info-json='[{\"chain-id\":1625,\"parent-chain-id\":1,\"chain-name\":\"conduit-orbit-deployer\",\"chain-config\":{\"chainId\":1625,\"homesteadBlock\":0,\"daoForkBlock\":null,\"daoForkSupport\":true,\"eip150Block\":0,\"eip150Hash\":\"0x0000000000000000000000000000000000000000000000000000000000000000\",\"eip155Block\":0,\"eip158Block\":0,\"byzantiumBlock\":0,\"constantinopleBlock\":0,\"petersburgBlock\":0,\"istanbulBlock\":0,\"muirGlacierBlock\":0,\"berlinBlock\":0,\"londonBlock\":0,\"clique\":{\"period\":0,\"epoch\":0},\"arbitrum\":{\"EnableArbOS\":true,\"AllowDebugPrecompiles\":false,\"DataAvailabilityCommittee\":true,\"InitialArbOSVersion\":11,\"InitialChainOwner\":\"0xd65776c5F9fA552cB5C9556B3e86bF6c376b233b\",\"GenesisBlockNum\":0}},\"rollup\":{\"bridge\":\"0x7983403dDA368AA7d67145a9b81c5c517F364c42\",\"inbox\":\"0x7AD2a94BefF3294a31894cFb5ba4206957a53c19\",\"sequencer-inbox\":\"0x8D99372612e8cFE7163B1a453831Bc40eAeb3cF3\",\"rollup\":\"0xf993AF239770932A0EDaB88B6A5ba3708Bd58239\",\"validator-utils\":\"0x2b0E04Dc90e3fA58165CB41E2834B44A56E766aF\",\"validator-wallet-creator\":\"0x9CAd81628aB7D8e239F1A5B497313341578c5F71\",\"deployed-at\":19898364}}]' \
  --http.api=net,web3,eth \
  --http.corsdomain=* \
  --http.addr=0.0.0.0 \
  --http.port=8547 \
  --http.vhosts=* \
  --node.data-availability.enable \
  --node.data-availability.rest-aggregator.enable \
  --node.data-availability.rest-aggregator.urls=https://das-gravity-mainnet-0.t.conduit.xyz \
  --execution.forwarding-target=https://rpc.gravity.xyz \
  --node.feed.input.url=wss://relay-gravity-mainnet-0.t.conduit.xyz \
  --parent-chain.blob-client.beacon-url=<ethereum_beacon_chain_rpc>



KillSignal=SIGHUP

[Install]
WantedBy=multi-user.target
```

{% hint style="info" %}
1. Run your own Ethereum mainnet node or use a node provider with unlimited rate limit for `eth_getLogs` and replace `<ethereum_mainnet_rpc>` with your rpc endpoint.
2. Replace `<ethereum_beacon_chain_rpc>` with your own Ethereum beacon chain rpc endpoint.
{% endhint %}

_**Ctrl+X and Y to save changes**_

```bash
systemctl enable gravity.service #enable gravity service at system startup

sudo systemctl daemon-reload #refresh systemd configuration when changes made

sudo systemctl start gravity.service #start gravity

sudo systemctl stop gravity.service #stop gravity
```

{% hint style="info" %}
To check or modify gravity`.service` parameters simply run&#x20;

`sudo nano /etc/systemd/system/`gravity`.service`

Ctrl+X and Y to save changes
{% endhint %}

```bash
journalctl -f -u gravity.service  #follow logs of gravity service
```

{% hint style="success" %}
{% code overflow="wrap" %}
```
The logs should look like below and indicate that your node syncs and is expected to reach chainhead in about a week

INFO [01-28|23:32:36.178] created block                            l2Block=36,676,753 l2BlockHash=f84024..8ee71c
INFO [01-28|23:32:37.179] created block                            l2Block=36,676,756 l2BlockHash=1e0d92..5f8afb
INFO [01-28|23:32:38.180] created block                            l2Block=36,676,760 l2BlockHash=e097e7..079758
```
{% endcode %}
{% endhint %}

## References

{% embed url="https://docs.gravity.xyz/network/run-a-gravity-alpha-mainnet-l2-node" %}

{% embed url="https://docs.arbitrum.io/node-running/how-tos/running-an-orbit-node" %}

{% embed url="https://docs.conduit.xyz/guides/run-a-node/arbitrum-node" %}
