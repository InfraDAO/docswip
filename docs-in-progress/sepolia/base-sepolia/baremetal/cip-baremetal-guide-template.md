# 💻 Op-Reth

Authors: \[ Ankur | Dapplooker]

## System Requirements

| CPU    | OS           | RAM   | DISK        |
| ------ | ------------ | ----- | ----------- |
| 8 vCPU | Ubuntu 22.04 | 16 GB | 1 TB (SSD)  |

{% hint style="success" %}
_The Madara node has a size of 585 GB on 14 , March, 2025._
{% endhint %}

## Pre-requisite <a href="#pre-requisite" id="pre-requisite"></a>

Before starting, clean the setup then update and upgrade. Install following:

* Git
* Go v1.23+
* rustc
* make
* just
* op-node&#x20;
* op-reth
* cargo
* jq&#x20;
* Ethereum Sepolia L1 RPC URL
* L1 Consensus Layer Beacon URL

{% hint style="warning" %}
### Before you start, make sure that you have your own synced `Ethereum Sepolia L1 RPC URL` and `L1 Consensus Layer Beacon endpoint` (e.g. Lighthouse Sepolia) ready . <a href="#before-you-start-make-sure-that-you-have-your-own-synced-ethereum-sepolia-l1-rpc-url-and-l1-consensu" id="before-you-start-make-sure-that-you-have-your-own-synced-ethereum-sepolia-l1-rpc-url-and-l1-consensu"></a>
{% endhint %}

### Installation Command

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install -y git make wget gcc pkg-config libusb-1.0-0-dev libudev-dev jq gcc g++ curl libssl-dev screen apache2-utils build-essential pkg-config
```

## Setting up Firewall <a href="#setting-up-firewall" id="setting-up-firewall"></a>

### Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow SSH

```bash
sudo ufw allow 22/tcp
sudo ufw allow 8546
sudo ufw allow 8547
```

### Allow Remote connection

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
```

### Enable Firewall

```bash
sudo ufw enable
```

#### &#x20; <a href="#install-docker" id="install-docker"></a>

## References

Share links to relevant docs and additional existing resources
