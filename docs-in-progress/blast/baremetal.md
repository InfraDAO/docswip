---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 💻 Baremetal

## System Requirements

|      CPU     |           OS           |      RAM     |           DISK          |
| :----------: | :--------------------: | :----------: | :---------------------: |
| 8+ cores CPU | Debian 12/Ubuntu 22.04 | => 16 GB RAM | 1.5TB+ (NVME preffered) |

{% hint style="info" %}
_The Blast archive node has a size of TB on May 23, 2024_
{% endhint %}

## Blast

{% hint style="success" %}
Blast is a fork of Optimism’s open-source [OP Stack](https://stack.optimism.io/). In this guide, Optimism's `op-geth` and `op-node`binaries are built from source to facilitate the node's installation.
{% endhint %}

{% hint style="warning" %}
Before you start, make sure that you have your own synced Ethereum L1 RPC URL (e.g. Erigon) and L1 Consensus Layer Beacon endpoint (e.g. Lighthouse) ready.
{% endhint %}

## Pre-Requisites

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y git make wget gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config
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

Allow remote RPC connections with Base Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

```bash
sudo ufw enable
```
