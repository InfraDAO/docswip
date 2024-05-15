---
description: 'Author: [man4ela | catapulta.eth]'
---

# 🐳 Linea Docker

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
WHITELIST={YOUR_REMOTE_MACHINE_IP} #the server's IP itself and comma separated IP's allowed to connect to RPC (e.g. Indexer)
</code></pre>

```wasm
{YOUR_DOMAIN} should look something like 117-230-108-65.bash-st.art
```

{% hint style="info" %}
ctrl + x and y to save file
{% endhint %}

### Make configuration directory

```
mkdir config
cd config
```

### Download genesis.json and rollup.json

```bash
curl -LO https://docs.linea.build/files/geth/mainnet/genesis.json
```

Create `linea-mainnet` docker volume

```
docker volume create linea-mainnet
```

### Initialize Geth

This command runs a Docker container using the`geth` image. It mounts two volumes: `linea-mainnet` to `/data` inside the container and `/root/linea/config` to `/config`. The container then initializes the Ethereum client with a genesis file located at `/config/genesis.json` using the `--datadir` option to specify the data directory as `/data`

```bash
docker run -v linea-mainnet:/data -v /root/linea/config:/config ethereum/client-go:v1.13.4 --datadir=/data init /config/genesis.json
```



{% hint style="success" %}
You should receive a quick output that your genesis file has been initialized.&#x20;

`INFO [05-15|01:03:37.079] Successfully wrote genesis state database=lightchaindata hash=b6762a..0ffbc6`
{% endhint %}

### Create docker-compose.yml

```bash
cd ~/linea

sudo nano docker-compose.yml
```

Paste the following into the docker-compose.yml

```bash
version: '3.8'

networks:
  monitor-net:
    driver: bridge

volumes:
    linea-mainnet: {}
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
      - "--certificatesresolvers.myresolver.acme.email=$EMAIL"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    volumes:
      - "traefik_letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.ipwhitelist.ipwhitelist.sourcerange=$WHITELIST"

######################################################################################
##################            LINEA GETH CONTAINER         ###########################
######################################################################################

  linea-mainnet:
    image: ethereum/client-go:v1.13.4
    container_name: linea-mainnet
    restart: unless-stopped
    networks:
      - monitor-net
    expose:
      - "8545"
      - "8546"
      - "7300"
    ports:
      - "30303:30303" # Peers
      - "30303:30303/udp" # Peers
    volumes:
      - ./config/genesis-l2.json:/mainnet/genesis-l2.json
      - linea-mainnet:/data
    command:
      - --datadir=/data
      - --networkid=59144
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
      - --http.api=web3,eth,txpool,net
      - --http.vhosts=*
      - --ws
      - --ws.addr=0.0.0.0
      - --ws.port=8546
      - --ws.origins=*
      - --ws.api=web3,eth,txpool,net
      - --bootnodes=enode://ca2f06aa93728e2883ff02b0c2076329e475fe667a48035b4f77711ea41a73cf6cb2ff232804c49538ad77794185d83295b57ddd2be79eefc50a9dd5c48bbb2e@3.23.106.165:30303,enode://eef91d714494a1ceb6e06e5ce96fe5d7d25d3701b2d2e68c042b33d5fa0e4bf134116e06947b3f40b0f22db08f104504dd2e5c790d8bcbb6bfb1b7f4f85313ec@3.133.179.213:30303,enode://cfd472842582c422c7c98b0f2d04c6bf21d1afb2c767f72b032f7ea89c03a7abdaf4855b7cb2dc9ae7509836064ba8d817572cf7421ba106ac87857836fa1d1b@3.145.12.13:30303
      - --syncmode=full
      - --metrics
      - --metrics.addr=0.0.0.0
      - --verbosity=3
      - --gcmode=archive

    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.linea-stripprefix.stripprefix.prefixes=/linea-mainnet"
      - "traefik.http.routers.linea.service=linea" #https
      - "traefik.http.services.linea.loadbalancer.server.port=8545"
      - "traefik.http.routers.linea.entrypoints=websecure"
      - "traefik.http.routers.linea.tls.certresolver=myresolver"
      - "traefik.http.routers.linea.rule=Host(`$DOMAIN`) && PathPrefix(`/linea-mainnet`)"
      - "traefik.http.routers.linea.middlewares=linea-stripprefix, ipwhitelist"
```

### Run Linea Node

<pre class="language-bash"><code class="lang-bash">cd ~/linea

<strong>docker-compose up -d
</strong></code></pre>

### Monitor Logs

Use `docker logs` to monitor your geth and op nodes. The `-f` flag ensures you are following the log output

```bash
docker logs linea-mainnet -f --tail 100
```

## Test Linea RPC

{% code overflow="wrap" %}
```bash
curl --data '{"method":"eth_syncing","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST https://{YOUR_DOMAIN}/linea-mainnet
```
{% endcode %}

The result will return `false` if a node is syncing
