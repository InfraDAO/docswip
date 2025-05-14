---
description: 'Authors: [man4ela | catapulta.eth]'
---

# Op-Erigon

## System Requirements

<table><thead><tr><th align="center">CPU</th><th align="center">OS</th><th width="254" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">8-Core CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 16 GB RAM</td><td align="center"><p>2 TB+</p><p> (NVMe)</p></td></tr></tbody></table>

{% hint style="info" %}
_Op-Erigon Base Sepolia archive node has a size of 1,5TB on May 14th, 2025_
{% endhint %}

{% hint style="success" %}
Base is a secure, low-cost Ethereum L2 built on Optimism’s open-source [OP Stack](https://stack.optimism.io/). In this guide, we cover docker installation of `op-erigon` and `op-node`to facilitate the node's synchronization on Sepolia Testnet Network. This method is expected to sync an archive node successfully in \~week using the snapshot provided by Testinprod-io
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

**Create Base Sepolia directory**

```bash
mkdir base-sepolia && cd base-sepolia
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

<pre class="language-bash"><code class="lang-bash"><strong>EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
</strong>DOMAIN={YOUR_DOMAIN} #Domain should be something like rpc.mywebsite.com, e.g. base-sepolia.infradao.org
WHITELIST={YOUR_REMOTE_MACHINE_IP} #the server's own IP and comma separated list of IP's allowed to connect to RPC (e.g. Indexer)
LAYER_1_RPC={YOUR_L1_RPC} #Your ready synced L1 Ethereum Sepolia node RPC endpoint
L1_BEACON={YOUR_L1_BEACON} #Your synced L1 CL (Consensus Layer) Beacon endpoint, e.g. Lighthouse (Prysm, Lodestar) Sepolia
</code></pre>

{% hint style="info" %}
`Ctrl + x` and `y` to save file
{% endhint %}

{% hint style="danger" %}
Make sure `L1_BEACON` endpoint provides full historical blobs data. Otherwise you won't be able to sync
{% endhint %}

### Create JWT secret file

```bash
mkdir -p /root/data/base-sepolia/op-erigon/ && cd /root/data/base-sepolia/op-erigon/

openssl rand -hex 32 | tr -d "\n" > "./jwt.hex"
```

### Optional/Recommended: Download Base Snapshot

{% hint style="success" %}
This is an optional step based on whether you want to sync the node from scratch or sync the node from a snapshot. Based on InfraDAO’s experience, we recommend downloading a snapshot and syncing the node from that snapshot. Syncing the node from scratch will take weeks while the op-erigon snapshot requires much shorter time to download, some time to extract, and about a week to sync from that point.
{% endhint %}

To sync from a snapshot, visit Testinprod-io Node Snapshots page to find a latest Base Sepolia archive for Op-Erigon: [https://snapshot.testinprod.io/](https://snapshot.testinprod.io/).

{% hint style="warning" %}
Testinprod doesn't update Base Sepolia snapshot regularly so expect about 3 weeks to sync a node from the most recent snapshot available
{% endhint %}

As downloading a snapshot takes some time it is good idea to run it in a screen session

```
screen -S erigon
```

Use `aria2c` to download the most recent **Base Sepolia Op-Erigon Archive** Snapshot

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>cd /root/base-sepolia
</strong>
aria2c --file-allocation=none -c -x 15 -s 15 https://datadirs.testinprod.io/base-sepolia-db-13833382.zst
</code></pre>

_press `ctrl+A and D` to return to previous screen and continue installation_

```bash
screen -r erigon #will bring you back to monitor downloading progress
```

You'll need to extract the downloaded snapshot and move its contents to the `op-erigon-data` directory, where Docker stores persistent data.

```bash
zstd -d base-sepolia-db-13833382.zst -c | tar xvf -
# replace the archive with an actual name
```

{% hint style="warning" %}
If you initially tried to sync the node from scratch and are now trying with a snapshot make sure to empty the destination directory first:
{% endhint %}

<pre class="language-bash"><code class="lang-bash"><strong>cd /var/lib/docker/volumes/base-sepolia_op-erigon_data/_data 
</strong>
ls 

#if directory isn't empty remove contents of chaindata directory

rm -rf ~/chaindata/*

mv /root/base-sepolia/mdbx.dat /var/lib/docker/volumes/base-sepolia_op-erigon_data/_data/chaindata

cd /var/lib/docker/volumes/base-sepolia_op-erigon_data/_data/chaindata

ls #to check if archive has moved properly
</code></pre>

{% hint style="warning" %}
If you haven't started the node yet, create `op-erigon-data directory first:`

```bash
docker volume create base-sepolia_op-erigon_data

cd /var/lib/docker/volumes/base-sepolia_op-erigon_data/_data

mv /root/base-sepolia/mdbx.dat /var/lib/docker/volumes/base-sepolia_op-erigon_data/_data/

ls
```
{% endhint %}

### Launch Base Sepolia

{% hint style="danger" %}
IMPORTANT:&#x20;

See avaialable up-to-date images for op-erigon here: [https://hub.docker.com/r/testinprod/op-erigon/tags](https://hub.docker.com/r/testinprod/op-erigon/tags)

for op-node here:  [https://github.com/ethereum-optimism/optimism/releases](https://github.com/ethereum-optimism/optimism/releases)
{% endhint %}

```bash
cd /root/base-sepolia

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
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:v1.13.0
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
      - /root/data/base-sepolia/op-erigon/jwt.hex:/root/data/base-sepolia/op-erigon/jwt.hex:ro
    environment:
      - OP_GETH_SEQUENCER_HTTP=https://sepolia-sequencer.base.org
      - OP_NODE_NETWORK=base-sepolia
      - OP_NODE_L1_ETH_RPC=${LAYER_1_RPC}
      - OP_NODE_L1_BEACON=${L1_BEACON}
      - OP_NODE_L2_ENGINE_AUTH=/root/data/base-sepolia/op-erigon/jwt.hex
      - OP_NODE_L2_ENGINE_RPC=http://op-erigon:8551
      - OP_NODE_LOG_LEVEL=info
      - OP_NODE_METRICS_ADDR=0.0.0.0
      - OP_NODE_METRICS_ENABLED=true
      - OP_NODE_METRICS_PORT=7373
      - OP_NODE_P2P_AGENT=base
      - OP_NODE_P2P_LISTEN_IP=0.0.0.0
      - OP_NODE_P2P_LISTEN_TCP_PORT=9222
      - OP_NODE_P2P_LISTEN_UDP_PORT=9222
      - OP_NODE_ROLLUP_LOAD_PROTOCOL_VERSIONS=false
      - OP_NODE_RPC_ADDR=0.0.0.0
      - OP_NODE_RPC_PORT=7545
      - OP_NODE_SNAPSHOT_LOG=/tmp/op-node-snapshot-log
      - OP_NODE_VERIFIER_L1_CONFS=4
      - OP_NODE_L1_TRUST_RPC=true
      - OP_NODE_P2P_BOOTNODES=enr:-J64QBwRIWAco7lv6jImSOjPU_W266lHXzpAS5YOh7WmgTyBZkgLgOwo_mxKJq3wz2XRbsoBItbv1dCyjIoNq67mFguGAYrTxM42gmlkgnY0gmlwhBLSsHKHb3BzdGFja4S0lAUAiXNlY3AyNTZrMaEDmoWSi8hcsRpQf2eJsNUx-sqv6fH4btmo2HsAzZFAKnKDdGNwgiQGg3VkcIIkBg,enr:-J64QFa3qMsONLGphfjEkeYyF6Jkil_jCuJmm7_a42ckZeUQGLVzrzstZNb1dgBp1GGx9bzImq5VxJLP-BaptZThGiWGAYrTytOvgmlkgnY0gmlwhGsV-zeHb3BzdGFja4S0lAUAiXNlY3AyNTZrMaEDahfSECTIS_cXyZ8IyNf4leANlZnrsMEWTkEYxf4GMCmDdGNwgiQGg3VkcIIkBg

  ######################################################################################
  #####################            OP-ERIGON CONTAINER            ######################
  ######################################################################################

  op-erigon:
    image: testinprod/op-erigon:v2.61.3-0.9.1
    container_name: op-erigon
    user: root  # Run as root
    restart: unless-stopped
    networks:
      - monitor-net
    expose:
      - "8545"  # RPC
      - "8546"  # WebSocket
      - "7300"  # Metrics
      - "8551"  # AuthRPC
    ports:
      - "30303:30309"      # Peers
      - "30303:30309/udp"  # Peers
      - "8551:8551"
      - "8545:8545"
    volumes:
      - /root/data/base-sepolia/op-erigon/jwt.hex:/root/data/base-sepolia/op-erigon/jwt.hex:ro
      - op-erigon_data:/data
    command:
      - --datadir=/data
      - --authrpc.jwtsecret=/root/data/base-sepolia/op-erigon/jwt.hex
      - --authrpc.addr=0.0.0.0
      - --authrpc.port=8551
      - --authrpc.vhosts=*
      - --http
      - --http.addr=0.0.0.0
      - --http.port=8545
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
      - --chain=base-sepolia
      - --db.size.limit=8TB
      - --nodiscover
      - --p2p.allowed-ports=30303,30304,30305,30306,30307,30308,30309
      - --rollup.sequencerhttp=https://sepolia.base.org
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.base.service=base"
      - "traefik.http.services.base.loadbalancer.server.port=8545"
      - "traefik.http.routers.base.entrypoints=websecure"
      - "traefik.http.routers.base.tls.certresolver=myresolver"
      - "traefik.http.routers.base.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.base.middlewares=ipwhitelist"

```

```bash
docker compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your **op-erigon** and **op-node**. The `-f` flag ensures you are following the log output

<pre class="language-bash"><code class="lang-bash">docker logs op-erigon -f --tail 100

<strong>docker logs opnode -f --tail 100
</strong></code></pre>

Once your Base Sepolia node starts syncing, the logs should look like this:

for **op-erigon**:

```bash
[INFO] [03-10|05:42:30.938] Received GetPayloadV3                    payloadId=8
[INFO] [03-10|05:42:30.941] stage running - force txs, with params   txs=10 notxpool=true
[INFO] [03-10|05:42:30.941] [2/7 MiningCreateBlock] Start mine       block=13876752 baseFee=1049 gasLimit=45000000
[INFO] [03-10|05:42:31.045] [7/7 MiningFinish] block ready for seal  block=13876752 transactions=10 gasUsed=3504306 gasLimit=45000000 difficulty=0
[INFO] [03-10|05:42:31.046] Built block                              hash=0xc8c3d2ade046637caa6dda5b521622e4b957f4358d1ad396e1c5da080210f6f6 height=13876752 txs=10 executionRequests=0 gas used %=7.787 time=107.772433ms
[INFO] [03-10|05:42:31.051] [NewPayload] Handling new payload        height=13876752 hash=0xc8c3d2ade046637caa6dda5b521622e4b957f4358d1ad396e1c5da080210f6f6
[INFO] [03-10|05:42:31.085] [updateForkchoice] Fork choice update: flushing in-memory state (built by previous newPayload)
[INFO] [03-10|05:42:31.137] RPC Daemon notified of new headers       from=13876751 to=13876752 amount=1 hash=0xc8c3d2ade046637caa6dda5b521622e4b957f4358d1ad396e1c5da080210f6f6 header sending=7.574µs log sending=250ns
[INFO] [03-10|05:42:31.137] head updated                             hash=0xc8c3d2ade046637caa6dda5b521622e4b957f4358d1ad396e1c5da080210f6f6 number=13876752
[INFO] [03-10|05:42:31.156] [ForkChoiceUpdated] BlockBuilder added   payload=9
[INFO] [03-10|05:42:31.156] Building block...
```

for **op-node**:

```bash
t=2025-03-10T02:52:20+0000 lvl=info msg="Sync progress" reason="new chain head block" 
l2_finalized=0xaefc5cf3494778144b7be0b0f60ec021f62be261fe436408ec444d5beb96f59b:22902892 l2_safe=0x68567e5cbacbb9d765746e6057e15c52b20b403131f8910467cc1566deb10a8d:22903391 l2_pending_safe=0x68567e5cbacbb9d765746e6057e15c52b20b403131f8910467cc1566deb10a8d:22903391 
l2_unsafe=0x676daa0eff1ab38bc20afeae069231b7cb870d3280a89a4a0aa756f05be9a361:22903426 l2_backup_unsafe=0x0000000000000000000000000000000000000000000000000000000000000000:0 l2_time=1741575140
t=2025-03-10T02:52:20+0000 lvl=info msg="successfully processed payload" ref=0x676daa0eff1ab38bc20afeae069231b7cb870d3280a89a4a0aa756f05be9a361:22903426 txs=92
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

### References

{% embed url="https://docs.base.org/chain/run-a-base-node" %}

{% embed url="https://github.com/testinprod-io/op-erigon" %}
