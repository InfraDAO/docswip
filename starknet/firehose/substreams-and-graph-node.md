---
description: 'Authors: [man4ela | catapulta.eth]'
---

# 🎏 Substreams and Graph Node

### Substreams-powered subgraphs

This article explains how to configure your Graph node to be able to sync substreams-powered subgraphs on Starknet network

{% hint style="info" %}
Since Starknet is not an EVM-compatible chain, you **must** configure your Graph Node to use **Firehose** and **Substreams**
{% endhint %}

In order to serve Substreams-powered subgraphs, Graph Node must be configured with a Substreams provider for the relevant network, as well as a Firehose to track the chain head. These providers can be configured via a `config.toml` file:

Assuming you use `Stakesquid's` docker setup for your indexing operations you need to navigate to graph-node configs directory and modify your `config.tmpl` file:

<pre class="language-bash"><code class="lang-bash">cd graphprotocol-mainnet-docker

cd graph-node-configs

<strong>nano config.tmpl
</strong></code></pre>

Append or update the following section:

```bash
[chains.${CHAIN_1_NAME}]
shard = "primary"
protocol = "substreams"
provider = [
  { label = "substreams", details = { type = "substreams", url = "${CHAIN_1_GRPC}", keys = "", features=["compression"] } }
]
```

Then set the Environment variables by modifying `.env` file with Firehose configs:

```bash
CHAIN_1_NAME="starknet-mainnet"
CHAIN_1_GRPC="http://firehose-endpoint:10016"
```

{% hint style="warning" %}
Keep in mind we have to tell graph-node to use Firehose (grpc), not JSON-RPC method as we normally do for EVM chains
{% endhint %}

### Disable Extended Block Checks (Important)

Firehose for Starknet does not support **extended blocks** by default. You must disable this check or your subgraph will fail to sync.

In your `compose-graphnode.yml` (inside the `index-node` service), add:

<pre class="language-bash"><code class="lang-bash"><strong>nano compose-graphnode.yml
</strong></code></pre>

```bash
GRAPH_NODE_FIREHOSE_DISABLE_EXTENDED_BLOCKS_FOR_CHAINS: starknet-mainnet
```

### Launch Graph-node:

```bash
bash start-essential --force-recreate
```

### References:

{% embed url="https://docs.substreams.dev/how-to-guides/sinks/subgraph/graph-out" %}
