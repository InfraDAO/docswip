---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 💻 Baremetal

## System Requirements

|                CPU               |           OS           |  RAM  |    DISK    |
| :------------------------------: | :--------------------: | :---: | :--------: |
| 8 Cores (Fastest per core speed) | Debian 12/Ubuntu 22.04 | 16 GB | 1TB+ (SSD) |

## Run a tracing node

{% hint style="success" %}
Geth's `debug` and `txpool` APIs and OpenEthereum's `trace` module provide non-standard RPC methods for getting a deeper insight into transaction processing. Supporting these RPC methods is important because many projects, such as [The Graph](https://thegraph.com/), rely on them to index blockchain data.



To use the supported RPC methods, you need to run a **tracing node.** This guide covers the steps on how to setup and sync a tracing node on Moonbeam.
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
```

Allow remote RPC connections with Moonbeam node (_The default port for parachains is `9944` and `9945` for the embedded relay chain_)

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 9944 9945
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

```bash
sudo ufw enable
```

## Building a Node on Moonbeam

Download the Latest Release Binary

Check [release binary](https://github.com/moonbeam-foundation/moonbeam/releases) page and take the following steps to download the latest version:

Create a directory to store the binary and chain data (you might need `sudo`)

```bash
mkdir /var/lib/moonbeam-data
```

Use `wget` to grab the latest [release binary](https://github.com/moonbeam-foundation/moonbeam/releases) and output it to the directory created in the previous step:

```bash
wget https://github.com/moonbeam-foundation/moonbeam/releases/download/v0.37.2/moonbeam \
-O /var/lib/moonbeam-data/moonbeam
```

