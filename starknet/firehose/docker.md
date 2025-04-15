---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 🐳 Docker

## System Requirements

<table><thead><tr><th align="center">CPU</th><th align="center">OS</th><th width="254" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4-Core CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 8 GB RAM</td><td align="center"><p>150 GB+</p><p> (SSD or NVMe)</p></td></tr></tbody></table>

{% hint style="info" %}
_Firehose poller for Starknet has a size of 60GB on April 14th, 2025_
{% endhint %}

{% hint style="success" %}
The Firehose instrumentation service is added to a node for efficient capture and simple storage of blockchain data.&#x20;

Firehose extracts, transforms and saves blockchain data in a highly performant file-based strategy. Blockchain developers can then access data extracted by Firehose through binary data streams.

This guide shows how to set up the **Firehose** poller for Starknet with **Substreams** support. We use two components:

**`firehose-core (`**&#x72;eader, relayer, merger, firehos&#x65;**`)`**\
&#xNAN;**`firehose-starknet (`**&#x53;tarknet-specific fetche&#x72;**`)`**
{% endhint %}

{% hint style="warning" %}
## This method of setting up Firehose for Starknet assumes that you have a your own synced Starknet mainnet full (or archive) node (URL endpoint available) and Ethereum Mainnet L1 endpoint ready. In this guide, you'll be able to configure and run the entire Firehose stack with a single `docker run` command
{% endhint %}

### Pre-Requisites <a href="#pre-requisties" id="pre-requisties"></a>

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
​
sudo apt install -y wget curl screen git ufw
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

**Create Starknet directory**

```bash
mkdir firehose && cd firehose
```

### Create Dockerfile.firehose-reader

```bash
sudo nano Dockerfile.firehose-reader
```

Paste and save:

```bash
FROM ghcr.io/streamingfast/firehose-core:v1.9.5 as core
FROM ghcr.io/streamingfast/firehose-starknet:6141dea as firestarknet

FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    ca-certificates curl vim jq tzdata \
    && rm -rf /var/lib/apt/lists/*

COPY --from=core /app/firecore /usr/local/bin/firecore
COPY --from=firestarknet /app/firestarknet /usr/local/bin/firestarknet

ENTRYPOINT ["firecore"]
```

#### Create .env file <a href="#create-.env-file" id="create-.env-file"></a>

```bash
sudo nano .env
```

Paste and save:

```bash
DOMAIN={YOUR_DOMAIN} #Domain should be something like rpc.mywebsite.com, e.g. firehose.infradao.org
EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
WHITELIST={YOUR_REMOTE_MACHINE_IP} #the server's own IP and comma separated list of IP's allowed to connect to RPC (e.g. Indexer)/0
STARKNET_RPC_URL={YOUR_STARKNET_RPC} # your synced Starknet Mainnet full node endpoint
L1_ETH_RPC_URL={YOUR_L1_RPC} #Your ready synced L1 Ethereum Mainnet node RPC endpoint
```

### Launch Firehose:

```bash
sudo nano docker-compose.yml
```

Paste the following into the `docker-compose.yml:`&#x20;

```bash
networks:
  monitor-net:
    driver: bridge

volumes:
  traefik_letsencrypt: {}

services:
  traefik:
    image: traefik:latest
    container_name: traefik
    restart: always
    ports:
      - "443:443"
    networks:
      - monitor-net
    command:
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

  reader:
    build:
      context: .
      dockerfile: Dockerfile.firehose-reader
    container_name: firehose-reader
    command:
      - "--config-file"
      - ""
      - start
      - reader-node
      - --common-one-block-store-url=file:///data/storage/one-blocks
      - --reader-node-path=firestarknet
      - --reader-node-arguments=fetch 0 --state-dir=/data/storage/reader-state --block-fetch-batch-size=1 --interval-between-fetch=1s --latest-block-retry-interval=10s --starknet-endpoints=${STARKNET_RPC_URL} --eth-endpoints=${L1_ETH_RPC_URL}
    environment:
      - STARKNET_RPC_URL=${STARKNET_RPC_URL}
      - L1_ETH_RPC_URL=${L1_ETH_RPC_URL}
    volumes:
      - ./firehose-data:/data
    networks:
      - monitor-net

  merger:
    image: ghcr.io/streamingfast/firehose-core:v1.9.5
    container_name: firehose-merger
    command:
      - "--config-file"
      - ""
      - start
      - merger
      - --common-one-block-store-url=file:///data/storage/one-blocks
      - --common-merged-blocks-store-url=file:///data/storage/merged-blocks
      - --common-forked-blocks-store-url=file:///data/storage/forked-blocks
    volumes:
      - ./firehose-data:/data
    networks:
      - monitor-net

  relayer:
    image: ghcr.io/streamingfast/firehose-core:v1.9.5
    container_name: firehose-relayer
    command:
      - "--config-file"
      - ""
      - start
      - relayer
      - --common-one-block-store-url=file:///data/storage/one-blocks
      - --relayer-source=reader:10010
    volumes:
      - ./firehose-data:/data
    networks:
      - monitor-net

  firehose:
    image: ghcr.io/streamingfast/firehose-core:v1.9.5
    container_name: firehose
    command:
      - "--config-file"
      - ""
      - start
      - firehose
      - --common-one-block-store-url=file:///data/storage/one-blocks
      - --common-merged-blocks-store-url=file:///data/storage/merged-blocks
      - --common-forked-blocks-store-url=file:///data/storage/forked-blocks
      - --common-live-blocks-addr=relayer:10014
      - --common-first-streamable-block=0
      - --advertise-chain-name=starknet-mainnet
      - --firehose-grpc-listen-addr=:10015
      - --substreams-tier1-grpc-listen-addr=:10016
    ports:
      - "10015:10015"
      - "10016:10016"
    volumes:
      - ./firehose-data:/data
    networks:
      - monitor-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.substreams.rule=Host(`${DOMAIN}`) && PathPrefix(`/substreams`)"
      - "traefik.http.routers.substreams.entrypoints=websecure"
      - "traefik.http.routers.substreams.tls.certresolver=myresolver"
      - "traefik.http.services.substreams.loadbalancer.server.port=10016"
      - "traefik.http.middlewares.substreams-stripprefix.stripprefix.prefixes=/substreams"
      - "traefik.http.routers.substreams.middlewares=substreams-stripprefix"
      - "traefik.http.middlewares.substreams-ipallowlist.ipallowlist.sourcerange=${WHITELIST}"
      - "traefik.http.routers.substreams.middlewares=substreams-ipallowlist,substreams-stripprefix"
```

#### Save and run"

```bash
sudo docker compose build reader
sudo docker compose up -d
```

### Monitor Logs

Use `docker logs` to monitor if all your Firehose components run as expected. The `-f` flag ensures you are following the log output

```bash
docker compose logs -f reader -n 100

docker compose logs -f merger -n 100

docker compose logs -f relayer -n 100

docker compose logs -f firehose -n 100
```

#### Once your Firehose starts syncing, the logs from `reader` are expected to look like this:

```log
firehose-reader  | {"severity":"INFO","timestamp":"2025-04-15T06:20:44.163567417Z","logger":"firestarknet","message":"saved cursor","filepath":"/data/storage/reader-state/cursor.json","last_fired_block":"1317766 (0x3b94a9045d5f192a21610c232eddf1714e8451c156f32fda5d1fec63cf66450)","lib":"1317765 (0x20e3f57c2c60e2fe90a1f84dd5c52d57bf8296d9e6ab0e67fa421be16519ba8)","block_count":3,"logging.googleapis.com/labels":{}}
firehose-reader  | {"severity":"INFO","timestamp":"2025-04-15T06:20:44.16361648Z","logger":"firestarknet","message":"about to fetch block","block_to_fetch":1317767,"delay":0,"logging.googleapis.com/labels":{}}
firehose-reader  | {"severity":"INFO","timestamp":"2025-04-15T06:20:44.163624815Z","logger":"firestarknet","message":"requesting block","block_num":1317767,"logging.googleapis.com/labels":{}}
firehose-reader  | {"severity":"INFO","timestamp":"2025-04-15T06:20:44.163682214Z","logger":"firestarknet","message":"fetching block","block_num":1317767,"logging.googleapis.com/labels":{}}
firehose-reader  | {"severity":"INFO","timestamp":"2025-04-15T06:20:44.164810011Z","logger":"firestarknet","message":"got latest block num","latest_block_num":1317766,"requested_block_num":1317767,"logging.googleapis.com/labels":{}}
firehose-reader  | {"severity":"INFO","timestamp":"2025-04-15T06:20:55.076626475Z","logger":"firestarknet","message":"fetching block","block_num":1317767,"logging.googleapis.com/labels":{}}
firehose-reader  | {"severity":"INFO","timestamp":"2025-04-15T06:20:55.07811861Z","logger":"firestarknet","message":"got latest block num","latest_block_num":1317766,"requested_block_num":1317767,"logging.googleapis.com/labels":{}}
firehose-reader  | {"severity":"INFO","timestamp":"2025-04-15T06:20:59.566063635Z","logger":"reader-node","message":"console reader stats","block_rate":"0.033 blocks/s (37625 total)","last_block":"#1317766 (0x3b94a9045d5f192a21610c232eddf1714e8451c156f32fda5d1fec63cf66450) @ 15 Apr 25 06:19 +0000","last_parent_block":"#1317765 (0x20e3f57c2c60e2fe90a1f84dd5c52d57bf8296d9e6ab0e67fa421be16519ba8)","lib":1317765,"logging.googleapis.com/labels":{}}
```

### The Best way to test if substreams work is to run an actual substream&#x20;

`Install the Substreams CLI:`

```bash
wget https://github.com/streamingfast/substreams/releases/download/v1.15.0/substreams_linux_x86_64.tar.gz
tar -xzf substreams_linux_x86_64.tar.gz
chmod +x substreams
sudo mv substreams /usr/local/bin/
```

`Then run the sample clock substream:`

```bash
substreams run \
  -e localhost:10016 \
  https://github.com/pinax-network/substreams/releases/download/blocks-v0.1.0/blocks-v0.1.0.spkg \
  map_blocks \
  -s -100 \
  --plaintext
```

#### The expected output:

```log
----------- BLOCK #47,500 (0x359f792ad8da03eef623004e4b7986cacc547054c425fb96de6bc10b405ff73) age=16790h2m41.168753013s ---------------
{
  "@module": "map_blocks",
  "@block": 47500,
  "@type": "sf.substreams.v1.Clock",
  "@data": {
    "id": "0x359f792ad8da03eef623004e4b7986cacc547054c425fb96de6bc10b405ff73",
    "number": "47500",
    "timestamp": "2023-05-01T16:06:35Z"
  }
}

----------- BLOCK #47,501 (0x36a51372e1b22af6d7640e14c78ce7300863e1101d008147a5c1d0097a02ce5) age=16789h59m6.211169907s ---------------
{
  "@module": "map_blocks",
  "@block": 47501,
  "@type": "sf.substreams.v1.Clock",
  "@data": {
    "id": "0x36a51372e1b22af6d7640e14c78ce7300863e1101d008147a5c1d0097a02ce5",
    "number": "47501",
    "timestamp": "2023-05-01T16:10:10Z"
  }
}
```

### Referenes:

{% embed url="https://firehose.streamingfast.io/introduction/firehose-overview" %}

{% embed url="https://github.com/streamingfast/firehose-starknet" %}

{% embed url="https://github.com/streamingfast/firehose-core" %}
