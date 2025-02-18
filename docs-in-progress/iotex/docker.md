# 🐳 Docker

Authors: \[ Ankur | Dapplooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>8 vCPU</td><td>Ubuntu 22.04</td><td>32 GB</td><td>2 TB  (SSD)</td></tr></tbody></table>

{% hint style="success" %}
_The IoTeX node has a size of  602 GB on 12, February, 2025._
{% endhint %}

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker
* Git
* Go v1.23+

### **Commands**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io git ufw -y jq -y
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
sudo ufw allow from ${REMOTE.HOST.IP} to any port 15014 # Ethereum JSON API
```

## Setup Instructions&#x20;

1. Set the environment&#x20;

```bash
mkdir -p ~/iotex
cd ~/iotex

export IOTEX_HOME=$~/iotex

mkdir -p $IOTEX_HOME/data
mkdir -p $IOTEX_HOME/log
mkdir -p $IOTEX_HOME/etc

curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/v2.1.2/config_mainnet.yaml > $IOTEX_HOME/etc/config.yaml
curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/v2.1.2/genesis_mainnet.yaml > $IOTEX_HOME/etc/genesis.yaml
curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/v2.1.2/trie.db.patch > $IOTEX_HOME/data/trie.db.patch
```

2. Edit \`config.yaml\` file&#x20;

Edit `$IOTEX_HOME/etc/config.yaml`, look for `externalHost` and `producerPrivKey`, uncomment the lines and fill in your external IP and private key. If you leave `producerPrivKey` empty, your node will be assgined with a random key.

3. Download Data&#x20;

```
curl -L https://t.iotex.me/mainnet-data-latest > $IOTEX_HOME/data.tar.gz
tar -xzf data.tar.gz
```

{% hint style="info" %}
1. &#x20;If you plan to run your node as a [gateway](https://github.com/iotexproject/iotex-bootstrap/tree/master#gateway), please use the snapshot with index data: [https://t.iotex.me/mainnet-data-with-idx-latest](https://t.iotex.me/mainnet-data-with-idx-latest) .
2. If you only want to sync chain data from 0 height without relaying on legacy delegate election data from Ethereum, you can setup legacy delegate election data with following command:

```
curl -L https://storage.googleapis.com/blockchain-golden/poll.mainnet.tar.gz > $IOTEX_HOME/poll.tar.gz; tar -xzf $IOTEX_HOME/poll.tar.gz --directory $IOTEX_HOME/data
```

3. If you want to sync the chain from 0 height and also fetching legacy delegate election data from Ethereum, please change the `gravityChainAPIs` in config.yaml to use your infura key with Ethereum archive mode supported or change the API endpoint to an Ethereum archive node which you can access.
{% endhint %}

4. Start the node&#x20;

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

## Monitoring

### Check Ports

Ensure that TCP ports `4689` and `8080` are open on your firewall and load balancer (if applicable). Additionally, if you intend to use the node as a gateway, make sure the following ports are open:

* `14014` for the IoTeX native gRPC API
* `15014` for the Ethereum JSON API
* `16014` for the Ethereum WebSocket

### Monitor Logs of Docker Container&#x20;

```bash
docker ps 
docker logs iotex
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

* [Git hub](https://github.com/iotexproject/iotex-bootstrap?tab=readme-ov-file#join-mainnet) : Repository for Archive Node Setup
* [IoTeX explorer](https://iotexscan.io/) : Block Explorer

