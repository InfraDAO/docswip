---
description: 'Author: [man4ela | catapulta.eth]'
---

# 🐳 Docker

_Last updated at date: 15 May 2024_

### System Requirements <a href="#system-requirements" id="system-requirements"></a>

| CPU                      | OS           | RAM   | DISK     |
| ------------------------ | ------------ | ----- | -------- |
| A fast CPU with 4+ cores | Ubuntu 22.04 | 16GB+ | >= 1.5TB |

{% hint style="info" %}
_Note: The Linea archive node consumes 1.5 TB of space on May 14.2024_
{% endhint %}

## 🔲 Linea

* [Official Docs](https://docs.linea.build/build-on-linea/run-a-node#step-3-1)

### Pre-requisites

Update, upgrade, and clean the system, and then firewall management (ufw), Docker, and the Git version control system.

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io docker-compose git ufw -y
```

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH, HTTP and HTTPS

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80
sudo ufw allow 443
```

Enable Firewall

```bash
sudo ufw enable
```

### Create Linea directory

The first command, `mkdir Linea`, will create a new directory named Linea in the current location. The second command, `cd Linea`, will change your current working directory to the newly created base directory. Now you are inside the base directory and can start storing docker-compose and related files in it.

```bash
mkdir Linea
cd Linea
```

### Create .env file

```bash
sudo nano .env
```

Paste the following into the file.

<pre class="language-bash"><code class="lang-bash"><strong>EMAIL={YOUR_EMAIL} #Your email to receive SSL renewal emails
</strong>DOMAIN={YOUR_DOMAIN} #Quickly register and query your free domain by entering curl -X PUT bash-st.art
WHITELIST={YOUR_REMOTE_MACHINE_IP} #the server's IP itself and comma separated list of IP's allowed to connect to RPC (e.g. Indexer)
</code></pre>

```wasm
{YOUR_DOMAIN} should look something like 117-230-108-65.bash-st.art
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

#### Make configuration directory

```bash
mkdir config

cd config
```

#### Download genesis.json

```bash
curl -LO https://docs.linea.build/files/geth/mainnet/genesis.json
```

### Create docker-compose.yml

_Return to Linea directory_

```bash
cd ~/linea
```

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
  linea-mainnet:
    name: "linea-mainnet"

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

  init:
    image: ethereum/client-go:v1.13.4
    container_name: linea-init
    command:
      - init
      - /genesis.json
    volumes:
      - ./config/genesis.json:/genesis.json:ro
      - linea-mainnet:/root/.ethereum

  node:
    image: ethereum/client-go:v1.13.4
    container_name: linea-mainnet
    restart: unless-stopped
    depends_on:
      init:
        condition: service_completed_successfully
    command:
      - --networkid=59144
      - --gcmode=archive
      - --syncmode=full
      - --rpc.allow-unprotected-txs
      - --txpool.accountqueue=50000
      - --txpool.globalqueue=50000
      - --txpool.globalslots=50000
      - --txpool.pricelimit=1000000
      - --txpool.pricebump=1
      - --txpool.nolocals
      - --http
      - --http.addr=0.0.0.0
      - --http.port=8545
      - --http.corsdomain=*
      - --http.api=admin,web3,eth,txpool,net
      - --http.vhosts=*
      - --ws
      - --ws.addr=0.0.0.0
      - --ws.port=8546
      - --ws.origins=*
      - --ws.api=web3,eth,txpool,net
      - --metrics
      - --metrics.addr=0.0.0.0
      - --metrics.port=6060
      - --bootnodes=enode://ca2f06aa93728e2883ff02b0c2076329e475fe667a48035b4f77711ea41a73cf6cb2ff232804c49538ad77794185d83295b57ddd2be79eefc50a9dd5c48bbb2e@3.23.106.165:30303,enode://eef91d714494a1ceb6e06e5ce96fe5d7d25d3701b2d2e68c042b33d5fa0e4bf134116e06947b3f40b0f22db08f104504dd2e5c790d8bcbb6bfb1b7f4f85313ec@3.133.179.213:30303,enode://cfd472842582c422c7c98b0f2d04c6bf21d1afb2c767f72b032f7ea89c03a7abdaf4855b7cb2dc9ae7509836064ba8d817572cf7421ba106ac87857836fa1d1b@3.145.12.13:30303
      - --verbosity=3
    ports:
      - "30303:30303"
      - "30303:30303/udp"
      - "8545:8545"
      - "8546:8546"
      - "6060:6060"
    volumes:
      - ./config/genesis.json:/genesis.json:ro
      - linea-mainnet:/root/.ethereum
    networks:
      - monitor-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.linea-stripprefix.stripprefix.prefixes=/linea-mainnet"
      - "traefik.http.routers.linea.service=linea"
      - "traefik.http.services.linea.loadbalancer.server.port=8545"
      - "traefik.http.routers.linea.entrypoints=websecure"
      - "traefik.http.routers.linea.tls.certresolver=myresolver"
      - "traefik.http.routers.linea.rule=Host(`${DOMAIN}`) && PathPrefix(`/linea-mainnet`)"
      - "traefik.http.routers.linea.middlewares=linea-stripprefix,ipwhitelist"

```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Run Linea Node

```bash
docker-compose up -d
```

### Monitor Logs

Use `docker logs` to monitor your Linea node. The `-f` flag ensures you are following the log output

```bash
docker logs linea-mainnet -f --tail 100
```

## Test Linea RPC

You can call the JSON-RPC API methods to confirm the node is running. For example, call [`eth_syncing`](https://besu.hyperledger.org/public-networks/reference/api#eth\_syncing) to return the synchronization status. For example the starting, current, and highest block, or `false` if not synchronizing (or if the head of the chain has been reached)

{% code overflow="wrap" %}
```bash

curl https://{YOUR_DOMAIN}/linea-mainnet \
        -X POST \
        -H "Content-Type: application/json" \
        -d '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
```
{% endcode %}
