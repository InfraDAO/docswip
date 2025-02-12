# 🐳 Docker

Authors: \[ Ankur | Dapplooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>4 vCPU</td><td>Ubuntu 22.04</td><td>16 GB</td><td>500 GB (SSD)</td></tr></tbody></table>

{% hint style="success" %}
_The IoTeX node has a size of   GB on 00, February , 2025._
{% endhint %}

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker
* Git
* Ethereum Mainnet  RPC
*

### **Commands**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io git ufw -y jq -y
```
{% endcode %}

## Firewall Settings

### Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Allow SSH, HTTP, and HTTPS

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### Allow Remote connection

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
```

## Setup Instructions&#x20;

```bash
docker run -d --restart on-failure --name iotex \
        -p 4689:4689 \
        -p 14014:14014 \
        -p 15014:15014 \
        -p 16014:16014 \
        -p 8080:8080 \
        -v=$IOTEX_HOME/data:/var/data:rw \
        -v=$IOTEX_HOME/log:/var/log:rw \
        -v=$IOTEX_HOME/etc/config.yaml:/etc/iotex/config_override.yaml:ro \
        -v=$IOTEX_HOME/etc/genesis.yaml:/etc/iotex/genesis.yaml:ro \
        iotex/iotex-core:v2.1.2 \
        iotex-server \
        -config-path=/etc/iotex/config_override.yaml \
        -genesis-path=/etc/iotex/genesis.yaml \
        -plugin=gateway
```

## Monitor Logs

Monitor Logs of Docker Container&#x20;

```bash
docker ps 
docker logs  iotex
```

## Sync Status

Run a query to check the latest synchronized L2 block:

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber",
"params":[],"id":83}' http://localhost:15014
```

Response should look like:

```json
{"jsonrpc":"2.0","id":83,"result":"0x2112b2d"}
```

## REFERENCES

* [Gravity Docs](https://docs.gravity.xyz/network/run-a-gravity-alpha-mainnet-l2-node) : Deployment Guide for&#x20;
* [Gravity Alpha Mainnet explorer](https://explorer.gravity.xyz/) : Block Explorer

