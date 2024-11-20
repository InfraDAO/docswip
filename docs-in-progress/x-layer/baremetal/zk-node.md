---
icon: arrows-to-dot
description: 'Authors: [man4ela | catapulta.eth]'
---

# ZK-node

## System Requirements

<table><thead><tr><th width="150" align="center">CPU</th><th align="center">OS</th><th width="192" align="center">RAM</th><th align="center">DISK</th></tr></thead><tbody><tr><td align="center">4+ cores CPU</td><td align="center">Debian 12/Ubuntu 22.04</td><td align="center">=> 16 GB RAM</td><td align="center">1TB+ (SSD or NVMe)</td></tr></tbody></table>

{% hint style="info" %}
_The X Layer archive node has a size of 560GB on November 18th, 2024_
{% endhint %}

## Setup XLayer Node

#### X Layer is an EVM-compatible Layer 2 network built with Polygon CDK, using Zero-Knowledge (ZK) technology to enhance Ethereum’s scalability, security, and efficiency

{% hint style="warning" %}
This guide covers the installation of X Layer Node (referred to as ZKNode), Synchronizer, ZKProver, and configuration of the State and Pool databases



NOTE: You will need Ethereum L1 RPC endpoint in order to sync X Layer node
{% endhint %}

## Pre-Requisties

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y

sudo apt install build-essential libbenchmark-dev libomp-dev libgmp-dev nlohmann-json3-dev postgresql libpqxx-dev libpqxx-doc nasm libsecp256k1-dev grpc-proto libsodium-dev libprotobuf-dev libssl-dev cmake libgrpc++-dev protobuf-compiler protobuf-compiler-grpc uuid-dev
```

{% hint style="info" %}
Important dependency note: You must install libpqxx version 6.4.5. If your distribution installs a newer version, please compile libpqxx 6.4.5 and install it manually instead
{% endhint %}

Use `dpkg` to check the Installed version of `libpqxx:`

```bash
dpkg -l | grep libpqxx
```

### Setting up Firewall

Set explicit default UFW rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow SSH

```bash
sudo ufw allow 22/tcp
```

Allow remote RPC connections with Blast Node

```bash
sudo ufw allow from ${REMOTE.HOST.IP} to any port 8545

sudo ufw allow 5432/tcp
```

{% hint style="warning" %}
Not advised to allow all or unknown IP address to RPC port
{% endhint %}

Enable Firewall

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw enable
</strong></code></pre>

To check the status of UFW and see the current rules

<pre class="language-bash"><code class="lang-bash"><strong>sudo ufw status verbose
</strong></code></pre>

### Install GO

{% hint style="info" %}
Go version 1.21+ is required to build ZKnode
{% endhint %}

```bash
sudo wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz && \
rm go1.21.6.linux-amd64.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin:/root/.local/bin' >> /root/.bashrc

source /root/.bashrc

#verify Go installation
go version
```

### Build ZKnode

```bash
mkdir && cd zknode

git clone --recurse-submodules https://github.com/okx/xlayer-node.git

cd xlayer-node

make build
```

Move the `xlayer-node` binary from the build directory to the designated directory for execution:

```bash
mkdir -p /root/zknode/{node,prover}

mv /root/zknode/xlayer-node/dist/xlayer-node /root/zknode/node/
```

Download `ZKnode` configuration and genesis.json file:

```bash
mkdir -p /root/zknode/node/config

wget https://raw.githubusercontent.com/okx/Deploy/main/setup/zknode/mainnet/config/node.config.toml -O /root/zknode/node/config/node.config.toml

wget https://raw.githubusercontent.com/okx/Deploy/refs/heads/main/mainnet/genesis.config.json -O /root/zknode/node/config/genesis.config.json
```

Carefully change the following parameters inside configuration file:

```bash
 sudo nano /root/zknode/node/config/node.config.toml
```

\[State.DB]&#x20;

Host = "xlayer-mainnet-state-db"

\#specify <mark style="background-color:blue;">**Host = "127.0.0.1"**</mark>

\[Pool.DB]&#x20;

Host = "xlayer-mainnet-pool-db"&#x20;

\#specify <mark style="background-color:blue;">**Host = "127.0.0.1"**</mark>

\[Etherman]&#x20;

<mark style="background-color:blue;">URL = ""</mark>&#x20;

\#input your synced **L1 Ethereum RPC** endpoint

\[Synchronizer]&#x20;

SyncInterval = "1s"&#x20;

SyncChunkSize = 100&#x20;

\#add a line <mark style="background-color:blue;">**L1SynchronizationMode = "sequential"**</mark>

\[MTClient]&#x20;

URI = "xlayer-mainnet-prover:50061"&#x20;

\#specify <mark style="background-color:blue;">URI =</mark> <mark style="background-color:blue;"></mark><mark style="background-color:blue;">**"127.0.0.1:50061"**</mark>

\[Executor]&#x20;

URI = "xlayer-mainnet-prover:50071"&#x20;

\#specify <mark style="background-color:blue;">URI =</mark> <mark style="background-color:blue;"></mark><mark style="background-color:blue;">**"127.0.0.1:50071"**</mark>

\[Metrics]&#x20;

Host = "0.0.0.0"&#x20;

\#specify <mark style="background-color:blue;">Host =</mark> <mark style="background-color:blue;"></mark><mark style="background-color:blue;">**"127.0.0.1"**</mark>

\[HashDB]&#x20;

Host = "xlayer-mainnet-state-db"&#x20;

\#specify <mark style="background-color:blue;">Host =</mark> <mark style="background-color:blue;"></mark><mark style="background-color:blue;">**"127.0.0.1"**</mark>

```bash
ctrl X + Y and enter #to save changes
```

### Compile and build ZKProver

```bash
cd /root/zknode/

git clone --recursive https://github.com/0xPolygonHermez/zkevm-prover.git

cd zkevm-prover
```

#### Download necessary files

Download **an archive (\~75GB)**. It's a good idea to have it running in the `screen`session:

<pre class="language-bash"><code class="lang-bash"><strong>./tools/download_archive.sh
</strong></code></pre>

The archive will take up an additional 115GB of space once extracted.

Consider to recompile the `protobufs`:

```bash
cd src/grpc
make
cd ../..
```

Run `make` to compile the main project:

```bash
make clean
make generate
make -j
```

{% hint style="warning" %}
The compilation process may appear to be stuck at some point, but sit back, relax, and wait till it completes the building
{% endhint %}

Test Vectors

```bash
./build/zkProver -c testvectors/config_runFile_BatchProof.json
```

#### Download configuration files:

```bash
wget -P /root/zknode/prover https://raw.githubusercontent.com/okx/Deploy/main/setup/zknode/mainnet/config/prover.config.json
```

#### Modify `prover.config.json`:

```bash
sudo nano /root/zknode/prover/prover.config.json
```

Replace

```bash
"databaseURL": "postgresql://prover_user:prover_pass@xlayer-mainnet-state-db:5432/prover_db",
```

with&#x20;

```bash
"databaseURL": "postgresql://prover_user:prover_pass@127.0.0.1:5432/prover_db",
```

### Create systemd files for the ZKEVM Node components

#### Zknode:

```bash
sudo nano /etc/systemd/system/xlayer-rpc.service
```

Paste the configs and save by entering `ctrl+X` and `Y+ENTER`:

```bash
[Unit]
Description=XLayer RPC Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/zknode/node/
ExecStart=/root/zknode/node/xlayer-node run \
   --network custom \
   --custom-network-file /root/zknode/node/config/genesis.config.json \
   --cfg /root/zknode/node/config/node.config.toml \
   --components rpc

KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target
```

#### &#x20;Synchronizer:

```bash
sudo nano /etc/systemd/system/xlayer-sync.service
```

```bash
[Unit]
Description=XLayer Sync Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/zknode/node/
ExecStart=/root/zknode/node/xlayer-node run \
   --network custom \
   --custom-network-file /root/zknode/node/config/genesis.config.json \
   --cfg /root/zknode/node/config/node.config.toml \
   --components synchronizer

KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target
```

#### and ZKProver:

```bash
sudo nano /etc/systemd/system/xlayer-prover.service
```

```bash
[Unit]
Description=XLayer Prover Service
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
Type=simple
Restart=on-failure
RestartSec=5
TimeoutSec=900
User=root
Nice=0
LimitNOFILE=200000
WorkingDirectory=/root/zknode/zkevm-prover/
ExecStart=/root/zknode/zkevm-prover/build/zkProver -c /root/zknode/prover/prover.config.json

KillSignal=SIGTERM

[Install]
WantedBy=multi-user.target
```

## Configure Postgresql for ZKProver

Make sure to be inside `/root/zknode/zkevm-prover/` directory:

```rescript
./tools/statedb/create_db.sh prover_db prover_user prover_pass
```

<details>

<summary>Example of expected output</summary>

```log
ProverDB database creation
Installing PostgreSQL...
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
postgresql is already the newest version (14+238).
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Creating database prover_db...
could not change directory to "/root/zknode/zkevm-prover": Permission denied
Creating user prover_user...
could not change directory to "/root/zknode/zkevm-prover": Permission denied
CREATE ROLE
could not change directory to "/root/zknode/zkevm-prover": Permission denied
GRANT
Creating table state.merkletree...
CREATE SCHEMA
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
INSERT 0 1
Done.
Example of connection string to use in the config.json file:
  "databaseURL": "postgresql://prover_user:prover_pass@127.0.0.1:5432/prover_db"
```

</details>

### Depending on your Postgresql version (14 or 15) update `postgresql.conf`:

```bash
sudo nano /etc/postgresql/14/main/postgresql.conf
```

Set <mark style="background-color:orange;">max\_connections = 500</mark>

Save, Quit and reload postgresql to apply changes by running:

```bash
sudo systemctl restart postgresql

```

## Launch X Layer node:

```bash
sudo systemctl daemon-reload

sudo systemctl start xlayer-prover
sudo systemctl start xlayer-rpc.service
sudo systemctl start xlayer-sync.service
```

### Monitor logs

```bash
sudo journalctl -fu xlayer-prover -n 250

sudo journalctl -fu xlayer-rpc.service -n 250

sudo journalctl -fu xlayer-sync.service -n 250
```

During the synchonization, you are expected to get following log messages:

ZKNode:

```log
Nov 19 06:19:24 xlayer-node[3963121]: 2024-11-19T06:19:24.558Z        INFO        state/l2block.go:280        new l2 block detected: number 43801, hash 0x697fcd84070802e874a83723d16a760fcf544bab23b9c2c3c1e8047b485b1c47        {"pid": 3963121, "version": "v0.4.1-2-g837ab57e"}
Nov 19 06:19:24 xlayer-node[3963121]: 2024-11-19T06:19:24.559Z        INFO        state/l2block.go:280        new l2 block detected: number 43802, hash 0x3fc86022298cdeafd950a55dba81bc30ac06d52c73c43298180839374f68964f        {"pid": 3963121, "version": "v0.4.1-2-g837ab57e"}
Nov 19 06:19:24 xlayer-node[3963121]: 2024-11-19T06:19:24.559Z        INFO        state/l2block.go:280        new l2 block detected: number 43803, hash 0xcec7e916fb50f140a3573e491df07369b309cbcc5120577bc366765b5dd4b039        {"pid": 3963121, "version": "v0.4.1-2-g837ab57e"}
Nov 19 06:19:24 xlayer-node[3963121]: 2024-11-19T06:19:24.586Z        INFO        jsonrpc/dynamic_gas_price_xlayer.go:84        Dynamic gas price update period is 10s        {"pid": 3963121, "version": "v0.4.1-2-g837ab57e"}
Nov 19 06:19:34 xlayer-node[3963121]: 2024-11-19T06:19:34.585Z        WARN        pool/pool.go:647        No suggested min gas price since: 2024-11-19 06:14:34.585576643 +0000 UTC        {"pid": 3963121, "version": "v0.4.1-2-g837ab57e"}
Nov 19 06:19:34 xlayer-node[3963121]: github.com/0xPolygonHermez/zkevm-node/pool.(*Pool).pollMinSuggestedGasPrice
Nov 19 06:19:34 xlayer-node[3963121]:         /root/zknode/xlayer-node/pool/pool.go:647
Nov 19 06:19:34 xlayer-node[3963121]: github.com/0xPolygonHermez/zkevm-node/pool.(*Pool).StartPollingMinSuggestedGasPrice.func1
Nov 19 06:19:34 xlayer-node[3963121]:         /root/zknode/xlayer-node/pool/pool.go:181
Nov 19 06:19:34 xlayer-node[3963121]: 2024-11-19T06:19:34.587Z        INFO        jsonrpc/dynamic_gas_price_xlayer.go:84        Dynamic gas price update period is 10s        {"pid": 3963121, "version": "v0.4.1-2-g837ab57e"}
```

ZKProver:

```log
Nov 19 05:11:49 zkProver[3961854]: 20241119_051149_849293 aa38631 4252c40 --> BINARY_EXECUTOR starting...
Nov 19 05:11:49 zkProver[3961854]: 20241119_051149_849299 aa38631 4252c40 --> BINARY_BUILD_FACTORS starting...
Nov 19 05:11:50 zkProver[3961854]: 20241119_051150_983649 aa38631 4252c40 <-- BINARY_BUILD_FACTORS done: 1.134332 s
Nov 19 05:11:50 zkProver[3961854]: 20241119_051150_983665 aa38631 4252c40 --> BINARY_BUILD_RESET starting...
Nov 19 05:11:51 zkProver[3961854]: 20241119_051151_169003 aa38631 4252c40 <-- BINARY_BUILD_RESET done: 0.185324 s
Nov 19 05:11:51 zkProver[3961854]: 20241119_051151_169019 aa38631 4252c40 <-- BINARY_EXECUTOR done: 1.319726 s
Nov 19 05:11:51 zkProver[3961854]: 20241119_051151_169102 aa38631 4252c40 <-- PROVER_CONSTRUCTOR done: 6.762991 s
Nov 19 05:11:51 zkProver[3961854]: 20241119_051151_169105 aa38631 4252c40 Launching HashDB server thread...
Nov 19 05:11:51 zkProver[3961854]: 20241119_051151_169150 aa38631 4252c40 Launching executor server thread...
Nov 19 05:11:51 zkProver[3961854]: 20241119_051151_170116 aa38631 37fe640 Executor server listening on 0.0.0.0:50071
Nov 19 05:11:51 zkProver[3961854]: 20241119_051151_170129 aa38631 3fff640 HashDB server listening on 0.0.0.0:50061
```

Synchronizer:

```log
Nov 19 06:13:10 xlayer-node[3963058]: 2024-11-19T06:13:10.713Z        INFO        synchronizer/synchronizer.go:701        Position: 8. New block. BlockNumber: 19549461. BlockHash: 0xfc7aeddc5a22c867253bde2c2dcb54ba3fc971b2d0ff60564944747db27608b8        {"pid": 3963058, "version": "v0.4.1-2-g837ab57e"}
Nov 19 06:13:10 xlayer-node[3963058]: 2024-11-19T06:13:10.713Z        INFO        synchronizer/synchronizer.go:627        Syncing block 19549461 of 21219852        {"pid": 3963058, "version": "v0.4.1-2-g837ab57e"}
Nov 19 06:13:10 xlayer-node[3963058]: 2024-11-19T06:13:10.713Z        INFO        synchronizer/synchronizer.go:628        Getting rollup info from block 19549461 to block 19549562        {"pid": 3963058, "version": "v0.4.1-2-g837ab57e"}
Nov 19 06:13:10 xlayer-node[3963058]: 2024-11-19T06:13:10.735Z        INFO        etherman/etherman_xlayer.go:1239        tx hash: 0x95b701bc4bad615ce9790bac36273f56f5b5a0404dd4384dcb7cabd60ad72496, msg form:0xAF9d27ffe4d51eD54AC8eEc78f2785D7E11E5ab1, to:0x2B0ee28D4D51bC9aDde5E58E295873F61F4a0507        {"pid": 3963058, "version": "v0.4.1-2-g837ab57e"}
Nov 19 06:13:10 xlayer-node[3963058]: 2024-11-19T06:13:10.883Z        INFO        etherman/etherman_xlayer.go:1239        tx hash: 0x40d7c22c8f0c9087778c392e9d41f31b058dfa355f55579153d45bc03e53f9b5, msg form:0xAF9d27ffe4d51eD54AC8eEc78f2785D7E11E5ab1, to:0x2B0ee28D4D51bC9aDde5E58E295873F61F4a0507        {"pid": 3963058, "version": "v0.4.1-2-g837ab57e"}
Nov 19 06:13:10 xlayer-node[3963058]: 2024-11-19T06:13:10.885Z        INFO        dataavailability/dataavailability.go:73        Try to get data from trusted sequencer for batches [14]        {"pid": 3963058, "version": "v0.4.1-2-g837ab57e"}
```

### Run _`curl`_ command in the terminal to check the status of your node

<pre class="language-bash"><code class="lang-bash"><strong>curl -H "Content-type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:8545
</strong></code></pre>

When it returns `false` then your node is fully synchronized with the network

### References

{% embed url="https://github.com/okx/Deploy" %}

{% embed url="https://github.com/okx/xlayer-node/tree/dev/config/environments/mainnet" %}

{% embed url="https://www.okx.com/ru/xlayer/docs/developer/setup-zknode/setup-local-zknode" %}
