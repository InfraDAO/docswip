---
description: 'Authors: [Godwin]'
---

# Op-Erigon

## System Requirements

<table><thead><tr><th align="center">CPU</th><th align="center">OS</th><th width="254" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">8-Core CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 16 GB RAM</td><td align="center"><p>2 TB+</p><p> (NVMe)</p></td></tr></tbody></table>

{% hint style="info" %}
_Op-Erigon Optimism Sepolia archive node has a size of 706GB on March 27th, 2025_
{% endhint %}

{% hint style="success" %}
In this guide, we cover docker installation of `op-erigon` and `op-node`to facilitate the node's synchronization on Sepolia Testnet Network. This method is expected to sync an archive node successfully in days or weeks using the snapshot provided by Testinprod-io
{% endhint %}

{% hint style="warning" %}
## Before you start, make sure that you have your own synced Ethereum Sepolia L1 RPC URL and L1 Consensus Layer Beacon endpoint (e.g. Lighthouse Sepolia) ready
{% endhint %}

### Pre-Requisites <a href="#pre-requisties" id="pre-requisties"></a>

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
​
sudo apt install -y wget curl screen git ufw zstd
```

### Setting up Firewall <a href="#setting-up-firewall" id="setting-up-firewall"></a>

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

### Enable Firewall

```bash
sudo ufw enable
```

## Install Docker

#### Run this command to remove any conflicting docker

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

#### Add Docker's official GPG key:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

#### Add the repository to ppt sources:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update
```

#### Install docker

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test docker is working
sudo docker run hello-world

#Install docker compose

sudo apt-get update
sudo apt-get install docker-compose-plugin

# Test the docker version
docker compose version
```

## Setting up a domain name to access RPC

Get the IP address of the host machine, you can use the following command in a terminal or command prompt

```bash
curl ifconfig.me
```

Set an A record for a domain, you need to access the domain's DNS settings and create an A record that points to the IP address of the host machine. This configuration allows users to reach your domain by resolving the domain name to the specific IP address associated with your host machine.

{% embed url="https://youtu.be/QcNBLSSn8Vg" %}

**Create Optimism Sepolia directory**

```bash
mkdir optimism-sepolia && cd optimism-sepolia
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

<pre class="language-bash"><code class="lang-bash"><strong>EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
</strong>DOMAIN={YOUR_DOMAIN} #Domain should be something like rpc.mywebsite.com, e.g. optimism-sepolia.infradao.org
WHITELIST={YOUR_REMOTE_MACHINE_IP} #the server's own IP and comma separated list of IP's allowed to connect to RPC (e.g. Indexer)
LAYER_1_RPC={YOUR_L1_RPC} #Your ready synced L1 Ethereum Sepolia node RPC endpoint
L1_BEACON={YOUR_L1_BEACON} #Your synced L1 CL (Consensus Layer) Beacon endpoint, e.g. Lighthouse (Prysm, Lodestar) Sepolia
</code></pre>

{% hint style="info" %}
`Ctrl + x` and `y` to save file
{% endhint %}

### Create JWT secret file

```bash
mkdir -p /root/data/optimism-sepolia/op-erigon/ && cd /root/data/optimism-sepolia/op-erigon/

openssl rand -hex 32 | tr -d "\n" > "./jwt.hex"
```

### Optional/Recommended: Download OptimismSnapshot

To sync from a snapshot, visit Testinprod-io Node Snapshots page to find a latest OptimismSepolia archive for Op-Erigon: [https://snapshot.testinprod.io/](https://snapshot.testinprod.io/).

As downloading a snapshot takes some time it is good idea to run it in a screen session

```
screen -S erigon
```

Use `aria2c` to download the most recent **Optimism Sepolia Op-Erigon Archive** Snapshot

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>cd /root/optimism-sepolia
</strong>
aria2c --file-allocation=none -c -x 15 -s 15 https://datadirs.testinprod.io/op-sepolia-db-24726201.zst
</code></pre>

_press `ctrl+A and D` to return to previous screen and continue installation_

```bash
screen -r erigon #will bring you back to monitor downloading progress
```

You'll need to extract the downloaded snapshot and move its contents to the `op-erigon-data` directory, where Docker stores persistent data.

```bash
# check sha256sum
sha256sum op-sepolia-db-24726201.zst
# decompress first
zstd --decompress op-sepolia-db-24726201.zst -o mdbx.dat

```

{% hint style="warning" %}
If you initially tried to sync the node from scratch and are now trying with a snapshot make sure to empty the destination directory first:
{% endhint %}

<pre class="language-bash"><code class="lang-bash"><strong>cd /var/lib/docker/volumes/op-sepolia-erigon_op-erigon_data/_data 
</strong>
ls 

#if directory isn't empty remove contents of chaindata directory

rm -rf ~/chaindata/*

mv /root/optimism-sepolia/mdbx.dat /var/lib/docker/volumes/op-sepolia-erigon_op-erigon_data/_data/chaindata

cd /var/lib/docker/volumes/op-sepolia-erigon_op-erigon_data/_data/chaindata

ls #to check if archive has moved properly
</code></pre>

{% hint style="warning" %}
If you haven't started the node yet, create `op-erigon-data directory first:`
{% endhint %}

```bash
docker volume create op-sepolia-erigon_op-erigon_data

cd /var/lib/docker/volumes/op-sepolia-erigon_op-erigon_data/_data

mv /root/optimism-sepolia/mdbx.dat /var/lib/docker/volumes/op-sepolia-erigon_op-erigon_data/_data/

ls
```



### Launch Optimism Sepolia

```bash
cd /root/optimism-sepolia

sudo nano docker-compose.yml
```

#### Copy/Paste into docker-compose.yml:

```bash
networks:
  monitor-net:
    driver: bridge

volumes:
  op-erigon_data: {}
  traefik_letsencrypt: {}

services:

  ######################################################################################
  #####################         TRAEFIK PROXY CONTAINER          #######################
  ######################################################################################

  traefik:
    image: traefik:latest
    container_name: traefik
    restart: always
    ports:
      - "443:443"
    networks:
      - monitor-net
    command:
      - "--api=true"
      - "--api.insecure=true"
      - "--api.dashboard=true"
      - "--log.level=DEBUG"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=${EMAIL}"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    volumes:
      - "traefik_letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.ipwhitelist.ipwhitelist.sourcerange=${WHITELIST}"

  ######################################################################################
  #####################            OP-NODE CONTAINER             #######################
  ######################################################################################

  opnode:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:v1.11.1
    container_name: opnode
    user: root  # Run as root
    networks:
      - monitor-net
    restart: unless-stopped
    expose:
      - "7545"  # RPC
      - "7373"  # METRICS
    ports:
      - "9222:9222"      # P2P TCP
      - "9222:9222/udp"  # P2P UDP
    volumes:
      - /root/data/op-sepolia/op-erigon/jwt.hex:/root/data/op-sepolia/op-erigon/jwt.hex:ro
    environment:
      - OP_GETH_SEQUENCER_HTTP=https://sepolia-sequencer.optimism.io
      - OP_NODE_NETWORK=op-sepolia
      - OP_NODE_L1_ETH_RPC=${LAYER_1_RPC}
      - OP_NODE_L1_BEACON=${LAYER_1_BEACON}
      - OP_NODE_L2_ENGINE_AUTH=/root/data/op-sepolia/op-erigon/jwt.hex
      - OP_NODE_L2_ENGINE_RPC=http://op-erigon:8551
      - OP_NODE_L2_ENGINE_KIND=erigon
      - OP_NODE_LOG_LEVEL=info
      - OP_NODE_METRICS_ADDR=0.0.0.0
      - OP_NODE_METRICS_ENABLED=true
      - OP_NODE_METRICS_PORT=7300
      - OP_NODE_P2P_LISTEN_IP=0.0.0.0
      - OP_NODE_P2P_LISTEN_TCP_PORT=9222
      - OP_NODE_P2P_LISTEN_UDP_PORT=9222
      - OP_NODE_ROLLUP_LOAD_PROTOCOL_VERSIONS=true
      - OP_NODE_RPC_ADDR=0.0.0.0
      - OP_NODE_RPC_PORT=7545
      - OP_NODE_SNAPSHOT_LOG=/tmp/op-node-snapshot-log
      - OP_NODE_VERIFIER_L1_CONFS=4
      - OP_NODE_L1_TRUST_RPC=true


  ######################################################################################
  #####################            OP-ERIGON CONTAINER            ######################
  ######################################################################################

  op-erigon:
    image: testinprod/op-erigon:v2.61.3-0.8.4
    container_name: op-erigon
    user: root  # Run as root
    restart: unless-stopped
    expose:
      - "8549"  # RPC
      - "8546"  # WebSocket
      - "7300"  # Metrics
      - "8551"  # AuthRPC
    ports:
      - "8549:8549"
      - "30303:30309"      # Peers
      - "30303:30309/udp"  # Peers
      - "8551:8551"
    volumes:
      - /root/data/op-sepolia/op-erigon/jwt.hex:/root/data/op-sepolia/op-erigon/jwt.hex:ro
      - op-erigon_data:/data
    command:
      - --datadir=/data
      - --authrpc.jwtsecret=/root/data/op-sepolia/op-erigon/jwt.hex
      - --authrpc.addr=0.0.0.0
      - --authrpc.port=8551
      - --authrpc.vhosts=*
      - --http
      - --http.addr=0.0.0.0
      - --http.port=8549
      - --http.compression
      - --http.vhosts=*
      - --http.corsdomain=*
      - --http.api=eth,debug,net,trace,web3,erigon
      - --private.api.addr=0.0.0.0:9095
      - --ws
      - --ws.compression
      - --metrics
      - --metrics.addr=0.0.0.0
      - --metrics.port=7300
      - --torrent.download.rate=1000mb
      - --torrent.port=42070
      - --rpc.returndata.limit=1000000
      - --txpool.gossip.disable=true
      - --chain=op-sepolia
      - --db.size.limit=8TB
      - --nodiscover
      - --p2p.allowed-ports=30303,30304,30305,30306,30307,30308,30309
      - --rollup.sequencerhttp=https://sepolia-sequencer.optimism.io
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.opsep.service=opsep"
      - "traefik.http.services.opsep.loadbalancer.server.port=8545"
      - "traefik.http.routers.opsep.entrypoints=websecure"
      - "traefik.http.routers.opsep.tls.certresolver=myresolver"
      - "traefik.http.routers.opsep.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.opsep.middlewares=ipwhitelist"

```

```bash
docker compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your **op-erigon** and **op-node**. The `-f` flag ensures you are following the log output

<pre class="language-bash"><code class="lang-bash">docker logs op-erigon -f --tail 100

<strong>docker logs opnode -f --tail 100
</strong></code></pre>

Once your Optimism Sepolia node starts syncing, the logs should look like this:

for **op-erigon**:

```bash
[INFO] [03-25|18:41:05.779] [ForkChoiceUpdated] BlockBuilder added   payload=3148
[INFO] [03-25|18:41:05.780] Building block... 
[INFO] [03-25|18:41:05.780] Received GetPayloadV3                    payloadId=3148
[INFO] [03-25|18:41:05.781] stage running - force txs, with params   txs=3 notxpool=true
[INFO] [03-25|18:41:05.781] [2/7 MiningCreateBlock] Start mine       block=24729349 baseFee=254 gasLimit=60000000
[INFO] [03-25|18:41:05.783] [7/7 MiningFinish] block ready for seal  block=24729349 transactions=3 gasUsed=282425 gasLimit=60000000 difficulty=0
[INFO] [03-25|18:41:05.784] Built block                              hash=0xe189bc2e86bd91b256bc01fc8f12ab169026b2d68f6fa51b85614e85b2ff2a8d height=24729349 txs=3 executionRequests=0 gas used %=0.471 time=4.175784ms
[INFO] [03-25|18:41:05.785] [NewPayload] Handling new payload        height=24729349 hash=0xe189bc2e86bd91b256bc01fc8f12ab169026b2d68f6fa51b85614e85b2ff2a8d
[INFO] [03-25|18:41:05.790] [updateForkchoice] Fork choice update: flushing in-memory state (built by previous newPayload) 
```

for **op-node**:

```bash
t=2025-03-25T18:43:31+0000 lvl=info msg="Inserted new L2 unsafe block" hash=0x95bd2cd3815aee11804d3fa4646a7c41ce92ca2e743529964cb95de8d58dfeb0 number=24731418 state_root=0x4d7ab5a5bf2f6fe0a4487381f8126b676a3699950482d076955098e7541752ec timestamp=1741265376 parent=0x8d20bb5340237a492a79f1627c7361cdb413ffd4c4dab317d98194e0265bc8a1 prev_randao=0xceccf87534423e468320438ca04b3aeb3960b5f4372976a3cae88251ea47c33c fee_recipient=0x4200000000000000000000000000000000000011 txs=5 build_time=13.793ms insert_time=21.446ms total_time=35.239ms mgas=0.585381 mgasps=16.61149517020851
t=2025-03-25T18:43:31+0000 lvl=info msg="generated attributes in payload queue" txs=18 timestamp=1741265378
t=2025-03-25T18:43:31+0000 lvl=info msg="Inserted new L2 unsafe block" hash=0x5cdb6b8613459f5453c06cecd43c0dafc34adf81a14bc31f9a38dc3dc2ed4414 number=24731419 state_root=0x74fe9f2d4cfe9ec323c29803ce0202030915b47b42cf518e3854e7ce5221e91a timestamp=1741265378 parent=0x95bd2cd3815aee11804d3fa4646a7c41ce92ca2e743529964cb95de8d58dfeb0 prev_randao=0xceccf87534423e468320438ca04b3aeb3960b5f4372976a3cae88251ea47c33c fee_recipient=0x4200000000000000000000000000000000000011 txs=18 build_time=23.451ms insert_time=33.543ms total_time=56.995ms mgas=3.069423 mgasps=53.854156828466955
```

{% code overflow="wrap" %}
```bash
curl --data '{"method":"eth_syncing","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST https://{YOUR_DOMAIN}
```
{% endcode %}

The result will return `false` if a node is fully synced

**Alternatively you can run**

```bash
curl -d '{"id":0,"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest",false]}' \
  -H "Content-Type: application/json" https://{YOUR_DOMAIN}
```

and it will return more details about syncing progress

{% hint style="warning" %}
Sync speed will be highly dependent on your Layer 1 RPC
{% endhint %}

{% embed url="https://github.com/testinprod-io/op-erigon" %}

{% embed url="https://docs.optimism.io/operators/node-operators/tutorials/run-node-from-source" %}

{% embed url="https://docs.optimism.io/operators/node-operators/tutorials/node-from-docker" %}
