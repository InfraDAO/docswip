# 🐳 Docker

Authors: \[ Ankur | DappLooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>4 - 8 vCPU</td><td>Ubuntu 22.04</td><td>16 GB</td><td>4 TiB SSD (NVME)</td></tr></tbody></table>

{% hint style="success" %}
_The Celo L2 node has a size of  2.2 TiB on  30, June, 2025._
{% endhint %}

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker

### **Commands**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io ufw -y jq -y
```
{% endcode %}

## Firewall Settings

### Check status & enable UFW&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw enable
</strong>sudo ufw status verbose
</code></pre>

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
sudo ufw allow from ${REMOTE.HOST.IP} to any port 9993
```



## Setup Instructions&#x20;

{% stepper %}
{% step %}
### Download Celo-L1 Data&#x20;

{% hint style="info" %}
You can skip this step if you have already running L1 node&#x20;
{% endhint %}

#### Setup Director

```bash
mkdir -p /root/celo-data/celo
mkdir -p /root/celo-data/celo-l2
```

#### Download Data

```bash
screen -S nodeConfiguration
wget https://storage.googleapis.com/cel2-rollup-files/celo/celo-mainnet-migrated-chaindata.tar.zst
```

#### Extract File

```bash
tar --zstd -xvf celo-mainnet-migrated-chaindata.tar.zst -C /root/celo-data/celo/ 
```

{% hint style="info" %}
Use ctrl + A + D to go back to your main terminal session.

`screen -r nodeConfiguration` to return to the screen window.
{% endhint %}
{% endstep %}

{% step %}
### Migrate L1 Data&#x20;

```bash
git clone https://github.com/celo-org/celo-l2-node-docker-compose.git
cd celo-l2-node-docker-compose
```

#### Migrate L1 Data to L2 Data&#x20;

{% hint style="info" %}
Run it in background or in screen will take few hours to complete.
{% endhint %}

```bash
./migrate.sh pre mainnet /root/celo-data/ /root/celo-data/celo-l2/
```
{% endstep %}

{% step %}
### Configure Environment file

```bash
cd celo-l2-node-docker-compose
cp mainnet.env .env
```

#### Edit .env file&#x20;

Find & update .env file with the following configuration.

```bash
nano .env

NODE_TYPE=archive
OP_GETH__SYNCMODE=full
HISTORICAL_RPC_DATADIR_PATH==/root/celo-data/
DATADIR_PATH==/root/celo-data/celo-l2/
```
{% endstep %}

{% step %}
### Start the Node

```bash
docker compose up -d --build
```
{% endstep %}
{% endstepper %}

## Monitoring

### Monitor Logs of Docker Container&#x20;

```bash
docker ps 
docker compose logs 
```

{% hint style="warning" %}
Might need to restart the container if snapshot download stuck for a longer duration. Total time required to sync is \~72 hr
{% endhint %}

## Sync Status

### Latest Block

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://localhost:8545
```

_Response should look like:_

```json
{"jsonrpc":"2.0","id":1,"result":"0x2f19521"}
```

## REFERENCES

{% embed url="https://docs.bnbchain.org/bnb-smart-chain/developers/node_operators/archive_node/" %}
Development Guide
{% endembed %}

{% embed url="https://testnet.bscscan.com/" %}
Testnet Block Explorer
{% endembed %}
