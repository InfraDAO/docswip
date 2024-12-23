---
description: 'Author(s): BK and man4ela'
---

# 🐳 Docker

## System Requirements

|                CPU               |           OS           |  RAM  |             DISK             |
| :------------------------------: | :--------------------: | :---: | :--------------------------: |
| 8 Cores (Fastest per core speed) | Debian 12/Ubuntu 22.04 | 16 GB | 2TB+ (SSD or NVME preffered) |

{% hint style="info" %}
_The Moonbeam tracing node has a size of 2TB on December 23, 2024_
{% endhint %}

## Run a tracing node

{% hint style="success" %}
Geth's `debug` and `txpool` APIs and OpenEthereum's `trace` module provide non-standard RPC methods for getting a deeper insight into transaction processing. Supporting these RPC methods is important because many projects, such as [The Graph](https://thegraph.com/), rely on them to index blockchain data.



To use the supported RPC methods, you need to run a **tracing node.** This guide covers the steps on how to setup and sync a tracing node on Moonbeam using Docker.
{% endhint %}

## Pre-Requisties

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install -y wget curl screen git ufw
```

## Setting up Firewall

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

### Create Moonbeam directory

The first command, `mkdir moonbeam`, will create a new directory named moonbeam in the current location. The second command, `cd moonbeam`, will change your current working directory to the newly created base directory. Now you are inside the base directory and can start storing docker-compose and related files in it.

```bash
mkdir moonbeam && cd moonbeam
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

<pre class="language-bash"><code class="lang-bash"><strong>EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
</strong>DOMAIN={YOUR_DOMAIN} #Domain should be something like rpc.mywebsite.com, e.g. moonbeam.infradao.org
WHITELIST={YOUR_REMOTE_MACHINE_IP} #the server's IP itself and comma separated list of IP's allowed to connect to RPC (e.g. Indexer)
</code></pre>

{% hint style="info" %}
`Ctrl + x` and `y` to save file
{% endhint %}

### Make Database directory and set necessary permissions

```bash
mkdir /var/lib/moonbeam-data/

sudo chown -R $(id -u):$(id -g) /var/lib/moonbeam-data
```

### Create docker-compose.yml

_Create and paste the following into the docker-compose.yml_

```bash
sudo nano docker-compose.yml
```

```bash
version: '3.9'

networks:
  monitor-net:
    driver: bridge

volumes:
  traefik_letsencrypt: {}
  moonbeam-data: {}

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

  moonbeam:
    image: moonbeamfoundation/moonbeam-tracing:v0.42.1-3400-latest
    container_name: moonbeam
    volumes:
      - moonbeam-data:/var/lib/moonbeam-data/
    restart: unless-stopped
    command:
      - --base-path=/var/lib/moonbeam-data
      - --chain=moonbeam
      - --name="InfraDAO B"
      - --rpc-cors="*"
      - --state-pruning=archive
      - --trie-cache-size=1073741824
      - --db-cache=16000
      - --ethapi=debug,trace,txpool
      - --wasm-runtime-overrides=/moonbeam/moonbeam-substitutes-tracing
      - --runtime-cache-size=64
      - --
      - --name="InfraDAO B (Embedded Relay)"
      - --unsafe-ws-external
      - --unsafe-rpc-external
    expose:
      - 9944 # rpc + ws parachain
      - 9945 # rpc + ws relay chain
      - 22057 # p2p parachain
      - 51555 # p2p relay chain
      - 9615 # prometheus parachain
      - 9616 # prometheus relay chain
    ports:
      - "22057:22057"
      - "51555:51555"
    networks:
      - monitor-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.moonbeam.service=moonbeam"
      - "traefik.http.services.moonbeam.loadbalancer.server.port=9944"
      - "traefik.http.routers.moonbeam.entrypoints=websecure"
      - "traefik.http.routers.moonbeam.tls.certresolver=myresolver"
      - "traefik.http.routers.moonbeam.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.moonbeam.middlewares=ipwhitelist"
```

