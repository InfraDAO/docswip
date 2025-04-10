# 🐳 Docker

Authors: \[ Ankur | Dapplooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>2 vCPU</td><td>Ubuntu 22.04</td><td>4 GB</td><td>200 GB </td></tr></tbody></table>

{% hint style="success" %}
_The Madara node has a size of  166 GB on 05 , March, 2025._
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
sudo ufw allow from ${REMOTE.HOST.IP} to any port 9944 
```

## Setup Instructions&#x20;

{% stepper %}
{% step %}
### Pull Docker Image

Pull the madara image from the github container registry (ghr):

```bash
docker pull ghcr.io/madara-alliance/madara:latest
docker tag ghcr.io/madara-alliance/madara:latest madara:latest
docker rmi ghcr.io/madara-alliance/madara:latest
```
{% endstep %}

{% step %}
### Start the Node

```bash
docker run -d \
  -p 9944:9944 \
  -v /root/madara:/tmp/madara \
  --name Madara \
  madara:latest \
  --name Madara \
  --full \
  --network mainnet \
  --rpc-external \
  --l1-endpoint https://ethereum-rpc.publicnode.com # Replace with your RPC_Endpoint
```

{% hint style="info" %}
Update  [https://ethereum-rpc.publicnode.com](https://ethereum-rpc.publicnode.com) with you Ethereum RPC endpoint
{% endhint %}
{% endstep %}
{% endstepper %}

## Monitoring

### Monitor Logs of Docker Container&#x20;

```bash
docker ps 
docker logs Madara
```

## Sync Status

### Latest Block

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0", "method":"starknet_blockNumber", "params":[], "id":1}' http://localhost:9944
```

_Response should look like:_

```json
{"jsonrpc":"2.0","result":1203395,"id":1}
```

### Syncing Status

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0", "method":"starknet_syncing", "params":[], "id":1}' http://localhost:9944 | jq
```

_Response should look like:_

```json
{
  "jsonrpc": "2.0",
  "result": {
    "starting_block_hash": "0x47c3637b57c2b079b93c61539950c17e868a28f46cdef28f88521067f21e943",
    "starting_block_num": 0,
    "current_block_hash": "0x2e0e03d1735d4c84e5caac50e7e090368a368a2ca307476fff199e68029f84a",
    "current_block_num": 1203387,
    "highest_block_hash": "0x2e0e03d1735d4c84e5caac50e7e090368a368a2ca307476fff199e68029f84a",
    "highest_block_num": 1203387
  },
  "id": 1
}
```

## REFERENCES



{% embed url="https://github.com/madara-alliance/madara?tab=readme-ov-file#run-with-docker" %}
Github Repository&#x20;
{% endembed %}

{% embed url="https://voyager.online/" %}
Block Explorer
{% endembed %}
