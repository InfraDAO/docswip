# 🐳 Docker

Authors: \[ Ankur | Dapplooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>8 vCPU</td><td>Ubuntu 22.04</td><td>32 GB</td><td>2 TB  (SSD)</td></tr></tbody></table>

{% hint style="success" %}
_The IoTeX node has a size of  \<size> GB on \<date>, February, 2025._
{% endhint %}

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker
* Git
* Go v1.23+
* aria2

### **Commands**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io git ufw -y jq -y aria2 -y
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
sudo ufw allow from ${REMOTE.HOST.IP} to any port 3060
```

## Download and Import Database Dump

{% stepper %}
{% step %}
### Download Snapshot

{% hint style="info" %}
To download the Latest Snapshot visit [https://en-backups.matterlabs.dev](https://en-backups.matterlabs.dev) and copy the desired link.\
Run the below command in `screen` as the download may take hours to day depending upon internet speed & snapshot size.
{% endhint %}

```bash
aria2c -c -x 16 -s 16 --dir=/path/to/download --out=latest_node_backup.sql.gz --file-allocation=none --timeout=600 --retry-wait=30 --max-tries=0 <snapshot_link>

# Example download link
aria2c -c -x 16 -s 16 --dir=~/zksync/backup/ --out=latest_node_backup.sql.gz --file-allocation=none --timeout=600 --retry-wait=30 --max-tries=0 https://en-backups.matterlabs.dev/external_node_backup_2024_12_13T05_05.sql.gz
```
{% endstep %}

{% step %}
### Import Database


{% endstep %}
{% endstepper %}



## Setup Instructions&#x20;

1.

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

