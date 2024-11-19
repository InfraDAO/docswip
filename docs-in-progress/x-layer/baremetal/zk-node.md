---
icon: arrows-to-dot
description: 'Authors: [man4ela | catapulta.eth]'
---

# ZK-node

## System Requirements

<table><thead><tr><th width="150" align="center">CPU</th><th align="center">OS</th><th width="192" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4+ cores CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 16 GB RAM</td><td align="center">500GB (SSD or NVMe)</td></tr></tbody></table>

{% hint style="info" %}
_The X Layer archive node has a size of 181GB on November 18th, 2024_
{% endhint %}

## Setup XLayer Node

#### X Layer is an EVM-compatible Layer 2 network built with Polygon CDK, using Zero-Knowledge (ZK) technology to enhance Ethereum’s scalability, security, and efficiency

{% hint style="warning" %}
This guide covers the installation of X Layer Node (referred to as ZKNode), Synchronizer, ZKProver, and configuration of the State and Pool databases



NOTE: You will need Ethereum L1 RPC endpoint in order to sync X Layer node
{% endhint %}

## Pre-Requisties

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y libgtest-dev libomp-dev libgmp-dev git make wget aria2 gcc pkg-config libusb-1.0-0-dev libudev-dev jq g++ curl libssl-dev screen apache2-utils build-essential
```

