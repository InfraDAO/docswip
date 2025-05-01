---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 🎏 Substreams and Graph Node

### Substreams-powered subgraphs

This article explains how to configure your Graph node to be able to sync substreams-powered subgraphs on Starknet network

Since Starknet isn't EVM chain we must configure our graph-node to use Firehose.

In order to serve Substreams-powered subgraphs, Graph Node must be configured with a Substreams provider for the relevant network, as well as a Firehose or RPC to track the chain head. These providers can be configured via a `config.toml` file:

Assuming you use `Stakesquid's` docker setup for your indexing operations you need to navigate to graph-node configs directory and modify your `config.tmpl` file:

```bash
cd graphprotocol-mainnet-docker

cd graph-node-configs

nano config.tmpl

[chains.${CHAIN_1_NAME}]
shard = "primary"
protocol = "substreams"
provider = [
  { label = "substreams", details = { type = "substreams", url = "${CHAIN_1_GRPC}", keys = "", features=["compression"] } }
]
```

Then modify `.env` file with Firehose configs:

```bash
CHAIN_1_NAME="starknet-mainnet"
CHAIN_1_GRPC="http://firehose-endpoint:10016"
```

Before applying the changes make sure to disable checks for Firehose Extended Block Details by specifying Starknet chain:

Inside `INDEX NODE CONTAINER` in `compose-graphnode.yml`:

<pre class="language-bash"><code class="lang-bash"><strong>nano compose-graphnode.yml
</strong></code></pre>

add a line:

```bash
GRAPH_NODE_FIREHOSE_DISABLE_EXTENDED_BLOCKS_FOR_CHAINS: starknet-mainnet
```

### Launch Graph-node:

```bash
bash start-essential --force-recreate
```

### Monitor logs
