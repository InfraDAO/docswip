# 🐳 Docker

Authors: \[ Ankur | Dapplooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>8 vCPU</td><td>Ubuntu 22.04</td><td>16 GB</td><td>500 GB </td></tr></tbody></table>

{% hint style="success" %}
_The Chapel node has a size of  \<size> GB on  , March, 2025._
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
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8575
```

## Setup Instructions&#x20;

{% stepper %}
{% step %}
### Create config file

Create project directory

```bash
mkdir -p /mnt/bsc/config /mnt/bsc/data/node
```

Create _config.toml_ file

```json
[Eth]
NetworkId = 97

[Eth.Miner]
GasCeil = 70000000
GasPrice = 1000000000
Recommit = 10000000000

[Eth.TxPool]
Locals = []
NoLocals = true
Journal = "transactions.rlp"
Rejournal = 3600000000000
PriceLimit = 1000000000
PriceBump = 10
AccountSlots = 16
GlobalSlots = 4096
AccountQueue = 64
GlobalQueue = 1024
Lifetime = 10800000000000

[Eth.GPO]
Blocks = 20
Percentile = 60
OracleThreshold = 1000

[Node]
DataDir = "node"
InsecureUnlockAllowed = false
IPCPath = "geth.ipc"
HTTPHost = "0.0.0.0"
HTTPPort = 8575
HTTPVirtualHosts = ["*"]
HTTPModules = ["debug","eth", "net", "web3", "txpool", "trace", "parlia", "geth"]
WSPort = 8576
WSModules = ["debug","eth", "net", "web3", "txpool", "trace", "parlia", "geth" ]

[Node.P2P]
MaxPeers = 100
NoDiscovery = false
TrustedNodes = []
StaticNodes = [
"enode://0637d1e62026e0c8685b1db0ca1c767c78c95c3fab64abc468d1a64b12ca4b530b46b8f80c915aec96f74f7ffc5999e8ad6d1484476f420f0c10e3d42361914b@52.199.214.252:30311",
"enode://df1e8eb59e42cad3c4551b2a53e31a7e55a2fdde1287babd1e94b0836550b489ba16c40932e4dacb16cba346bd442c432265a299c4aca63ee7bb0f832b9f45eb@52.51.80.128:30311",
"enode://dbcc5ec23bdf89243688321e8cfa8d80e17edce093206bcc6df998d8148385767cae3058a1c1e20c93c3b8e07962bc7a321deab0aa46c106283f1220f12c220a@3.209.122.123:30311",
"enode://665cf77ca26a8421cfe61a52ac312958308d4912e78ce8e0f61d6902e4494d4cc38f9b0dd1b23a427a7a5734e27e5d9729231426b06bb9c73b56a142f83f6b68@52.72.123.113:30311"
]

[Node.LogConfig]
FileRoot = ""
FilePath = "bsc.log"
MaxBytesSize = 10485760
Level = "info"
```

{% hint style="info" %}
File config might change, look for updated code in docker image.&#x20;
{% endhint %}
{% endstep %}

{% step %}
### Pull Docker Image

Pull the image from the GitHub container registry (ghr):

```bash
docker pull ghcr.io/bnb-chain/bsc:1.5.7 
```

{% hint style="info" %}
Get latest version: [https://github.com/bnb-chain/bsc/pkgs/container/bsc](https://github.com/bnb-chain/bsc/pkgs/container/bsc)
{% endhint %}
{% endstep %}

{% step %}
### Start the Node

```bash
docker run -d \
  --user root \
  -v /mnt/bsc/config:/bsc/config \
  -v /mnt/bsc/data/node:/bsc/node \
  -p 8575:8575 \
  -p 8546:8546 \
  -p 30311:30311 \
  -p 30311:30311/udp \
  --restart on-failure \
  --name bsc \
  ghcr.io/bnb-chain/bsc:1.5.7 \
  --syncmode full \
  --gcmode archive \
  --history.transactions 0 \
  --txlookuplimit 0 \
  --cache 8000 \
  --http \
  --http.addr 0.0.0.0 \
  --http.port 8575 \
  --http.vhosts '*' \
  --http.corsdomain '*' \
  --http.api eth,net,web3,debug,txpool,trace,geth,parlia \
  --ws \
  --ws.addr 0.0.0.0 \
  --ws.port 8546 \
  --ws.origins '*' \
  --ws.api eth,net,web3,debug,txpool,trace,geth,parlia \
  --maxpeers 200 \
  --verbosity 5
```
{% endstep %}
{% endstepper %}

## Monitoring

### Monitor Logs of Docker Container&#x20;

```bash
docker ps 
```

## Sync Status

### Start geth console&#x20;

```bash
geth attach http://localhost:8575
eth.syncing        # Return block number no cervertion needed
eth.syncing.currentBlock # Return current highestblock synced
```

_results_

```json
{
  currentBlock: 8102900,
  healedBytecodeBytes: "0x0",
  healedBytecodes: "0x0",
  healedTrienodeBytes: "0x0",
  healedTrienodes: "0x0",
  healingBytecode: "0x0",
  healingTrienodes: "0x0",
  highestBlock: 49156874,
  startingBlock: 0,
  syncedAccountBytes: "0x0",
  syncedAccounts: "0x0",
  syncedBytecodeBytes: "0x0",
  syncedBytecodes: "0x0",
  syncedStorage: "0x0",
  syncedStorageBytes: "0x0",
  txIndexFinishedBlocks: "0x7b9f81",
  txIndexRemainingBlocks: "0x0"
}
```

### Latest Block

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://localhost:8575
```

_Response should look like:_

```json
{"jsonrpc":"2.0","id":1,"result":"0x7f5659"}
```

## REFERENCES

{% embed url="https://docs.bnbchain.org/bnb-smart-chain/developers/node_operators/archive_node/" %}
Development Guide
{% endembed %}

{% embed url="https://testnet.bscscan.com/" %}
Testnet Block Explorer
{% endembed %}
