# 🐳 Docker

Authors: \[ Ankur | Dapplooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>4 vCPU</td><td>Ubuntu 22.04</td><td>16 GB</td><td>2TB (SSD)</td></tr></tbody></table>

{% hint style="info" %}
_The zkEVM archival node has a size of 810 GB on January 16, 2025._
{% endhint %}

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker
* Docker Compose
* Git
* Unzip
* L1 Ethereum node RPC

### **Commands**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io docker-compose git ufw -y unzip -y 
```
{% endcode %}

## Firewall Settings

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
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545
```

## Setup Instructions&#x20;

1. **Set the variables** :

```bash
# define the network
ZKEVM_NET=mainet
# define installation path
ZKEVM_DIR=./mainnet
# define your config directory
ZKEVM_CONFIG_DIR=./mainnet
```

2. **Download and extract zkEVM artifacts** :

```bash
curl -L https://github.com/0xPolygonHermez/zkevm-node/releases/latest/download/$ZKEVM_NET.zip > $ZKEVM_NET.zip && unzip -o $ZKEVM_NET.zip -d $ZKEVM_DIR && rm $ZKEVM_NET.zip
```

3. **Copy the example environment file  :**

```bash
cp $ZKEVM_DIR/$ZKEVM_NET/example.env $ZKEVM_CONFIG_DIR/.env
```

4. Update the following variables in .env file :

```bash
# Network configuration
ZKEVM_NETWORK=mainnet
# L1 node URL
ZKEVM_NODE_ETHERMAN_URL="" #Etherium node URL
# PATH WHERE THE STATEDB POSTGRES CONTAINER WILL STORE PERSISTENT DATA
ZKEVM_NODE_STATEDB_DATA_DIR="/mainnet/container/"
# PATH WHERE THE POOLDB POSTGRES CONTAINER WILL STORE PERSISTENT DATA
ZKEVM_NODE_POOLDB_DATA_DIR="/mainnet/postgress/"
```

### Example docker compose file:

```yaml
version: "3.5"
networks:
  default:
    name: zkevm
services:
  zkevm-rpc:
    container_name: zkevm-rpc
    restart: unless-stopped
    depends_on:
      zkevm-pool-db:
        condition: service_healthy
      zkevm-state-db:
        condition: service_healthy
      zkevm-sync:
        condition: service_started
    image: hermeznetwork/zkevm-node:v0.7.3
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M
    ports:
      - 8545:8545
      - 9091:9091 # needed if metrics enabled
    environment:
      - ZKEVM_NODE_ETHERMAN_URL=${ZKEVM_NODE_ETHERMAN_URL}
    volumes:
      - ${ZKEVM_ADVANCED_CONFIG_DIR:-./config/environments/mainnet}/node.config.toml:/app/config.toml
    command:
      - "/bin/sh"
      - "-c"
      - "/app/zkevm-node run --network ${ZKEVM_NETWORK} --cfg /app/config.toml --components rpc"

  zkevm-sync:
    container_name: zkevm-sync
    restart: unless-stopped
    depends_on:
      zkevm-state-db:
        condition: service_healthy
    image: hermeznetwork/zkevm-node:v0.7.3
    ports:
      - 9092:9091 # needed if metrics enabled
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M
    environment:
      - ZKEVM_NODE_ETHERMAN_URL=${ZKEVM_NODE_ETHERMAN_URL}
    volumes:
      - ${ZKEVM_ADVANCED_CONFIG_DIR:-./config/environments/mainnet}/node.config.toml:/app/config.toml
    command:
      - "/bin/sh"
      - "-c"
      - "/app/zkevm-node run --network ${ZKEVM_NETWORK} --cfg /app/config.toml --components synchronizer"

  zkevm-state-db:
    container_name: zkevm-state-db
    restart: unless-stopped
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    ports:
      - 5432:5432
    volumes:
      - ./db/scripts/init_prover_db.sql:/docker-entrypoint-initdb.d/init.sql
      - ${ZKEVM_NODE_STATEDB_DATA_DIR}:/var/lib/postgresql/data
      - ${ZKEVM_ADVANCED_CONFIG_DIR:-./config/environments/mainnet}/postgresql.conf:/etc/postgresql.conf
    environment:
      - POSTGRES_USER=state_user
      - POSTGRES_PASSWORD=state_password
      - POSTGRES_DB=state_db
    command:
      - "postgres"
      - "-N"
      - "500"
      - "-c"
      - "config_file=/etc/postgresql.conf"

  zkevm-pool-db:
    container_name: zkevm-pool-db
    restart: unless-stopped
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    ports:
      - 5433:5432
    volumes:
      - ${ZKEVM_NODE_POOLDB_DATA_DIR}:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=pool_user
      - POSTGRES_PASSWORD=pool_password
      - POSTGRES_DB=pool_db
    command:
      - "postgres"
      - "-N"
      - "500"

  zkevm-prover:
    container_name: zkevm-prover
    restart: unless-stopped
    image: hermeznetwork/zkevm-prover:v6.0.0
    depends_on:
      zkevm-state-db:
        condition: service_healthy
    ports:
      - 50061:50061 # MT
      - 50071:50071 # Executor
    volumes:
      - ${ZKEVM_ADVANCED_CONFIG_DIR:-./config/environments/mainnet}/prover.config.json:/usr/src/app/config.json
    command: >
      zkProver -c /usr/src/app/config.json
```

4. Start the Node:&#x20;

```bash
sudo docker compose --env-file $ZKEVM_CONFIG_DIR/.env -f $ZKEVM_DIR/$ZKEVM_NET/docker-compose.yml up -d
```

## Monitor Logs

Monitor Logs of Docker Container&#x20;

```bash
docker ps 
docker logs <container-name>
```

{% hint style="info" %}
You should see the following containers&#x20;

* zkevm-rpc
* zkevm-sync
* zkevm-state-db
* zkevm-pool-db
* zkevm-prover
{% endhint %}

## Sync Status

Run a query to check the latest synchronized L2 block:

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber",
"params":[],"id":83}' http://localhost:8545
```

Response should look like:

```json
{"jsonrpc":"2.0","id":83,"result":"0x12565d8"}
```

## REFERENCES

* [GitHub Repository](https://github.com/0xPolygonHermez/zkevm-node) : Git Hub repository of zkEVM
* [Block Explorer](https://zkevm.polygonscan.com/) : zkEVM block explorer
