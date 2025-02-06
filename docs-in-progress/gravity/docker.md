# 🐳 Docker

Authors: \[ Ankur | Dapplooker]

## System Requirements

<table data-full-width="false"><thead><tr><th>CPU</th><th>OS</th><th>RAM</th><th>DISK</th></tr></thead><tbody><tr><td>4 vCPU</td><td>Ubuntu 22.04</td><td>16 GB</td><td>500 GB (SSD)</td></tr></tbody></table>

{% hint style="success" %}
_The Gravity node has a size of  <mark style="color:red;">\<Size></mark> GB on <mark style="color:red;">February , 2025.</mark>_
{% endhint %}

## Pre-requisite

Before starting, clean the setup then update and upgrade. Install following:

* Docker
* Git
* Go v1.23+
* Ethereum Mainnet  RPC
* Ethereum Beacon Chain RPC

### **Commands**

{% code overflow="wrap" %}
```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt auto-remove -y
sudo apt install docker.io git ufw -y 
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

```bash
docker run -d --name gravity_alpha_mainnet \
    -v ~/gravity/arbitrum:/home/user/.arbitrum \
    -p 0.0.0.0:8547:8547 \
    -p 0.0.0.0:8548:8548 \
    offchainlabs/nitro-node:v2.3.3-6a1c1a7 \
    --parent-chain.connection.url=<ethereum_mainnet_rpc> \
    --chain.id=1625 \
    --chain.name=conduit-orbit-deployer \
    --http.api=net,web3,eth \
    --http.corsdomain="*" \
    --http.addr=0.0.0.0 \
    --http.vhosts="*" \
    --chain.info-json='[
        {
            "chain-id":1625,
            "parent-chain-id":1,
            "chain-name":"conduit-orbit-deployer",
            "chain-config":{
                "chainId":1625,
                "homesteadBlock":0,
                "daoForkBlock":null,
                "daoForkSupport":true,
                "eip150Block":0,
                "eip150Hash":"0x0000000000000000000000000000000000000000000000000000000000000000",
                "eip155Block":0,
                "eip158Block":0,
                "byzantiumBlock":0,
                "constantinopleBlock":0,
                "petersburgBlock":0,
                "istanbulBlock":0,
                "muirGlacierBlock":0,
                "berlinBlock":0,
                "londonBlock":0,
                "clique":{"period":0,"epoch":0},
                "arbitrum":{
                    "EnableArbOS":true,
                    "AllowDebugPrecompiles":false,
                    "DataAvailabilityCommittee":true,
                    "InitialArbOSVersion":11,
                    "InitialChainOwner":"0xd65776c5F9fA552cB5C9556B3e86bF6c376b233b",
                    "GenesisBlockNum":0
                }
            },
            "rollup":{
                "bridge":"0x7983403dDA368AA7d67145a9b81c5c517F364c42",
                "inbox":"0x7AD2a94BefF3294a31894cFb5ba4206957a53c19",
                "sequencer-inbox":"0x8D99372612e8cFE7163B1a453831Bc40eAeb3cF3",
                "rollup":"0xf993AF239770932A0EDaB88B6A5ba3708Bd58239",
                "validator-utils":"0x2b0E04Dc90e3fA58165CB41E2834B44A56E766aF",
                "validator-wallet-creator":"0x9CAd81628aB7D8e239F1A5B497313341578c5F71",
                "deployed-at":19898364
            }
        }
    ]' \
    --node.data-availability.enable \
    --node.data-availability.rest-aggregator.enable \
    --node.data-availability.rest-aggregator.urls=https://das-gravity-mainnet-0.t.conduit.xyz \
    --execution.forwarding-target=https://rpc.gravity.xyz \
    --node.feed.input.url=wss://relay-gravity-mainnet-0.t.conduit.xyz \
    --parent-chain.blob-client.beacon-url=<ethereum_beacon_chain_rpc>
    --validation.wasm.enable-wasmroots-check=false
```

{% hint style="info" %}
<mark style="color:red;">Note</mark>&#x20;

The above command, we are using chain name and chain info obtained from conduit by:

`curl https://api.conduit.xyz/file/v1/arbitrum/chaininfo/gravity-mainnet-0`

Relay endpoint is provided by Conduit as `wss://relay-gravity-mainnet-0.t.conduit.xyz`

Gravity Alpha Mainnet is using Anytrust DA, the aggregator URL is:

`https://das-gravity-mainnet-0.t.conduit.xyz`
{% endhint %}

## Monitor Logs

Monitor Logs of Docker Container&#x20;

```bash
docker ps 
docker logs  gravity_alpha_mainet
```

## Sync Status

Run a query to check the latest synchronized L2 block:

```bash
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber",
"params":[],"id":83}' http://localhost:8547
```

Response should look like:

```json
{"jsonrpc":"2.0","id":83,"result":"0x78811a"}
```

## REFERENCES

* [Gravity Docs](https://docs.gravity.xyz/network/run-a-gravity-alpha-mainnet-l2-node) : Deployment Guide for Alpha Mainnet
* [Gravity Alpha Mainnet explorer](https://explorer.gravity.xyz/) : Block Explorer

