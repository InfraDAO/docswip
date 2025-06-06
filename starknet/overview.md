# 🔍 Overview

By: Leo & Kate

[**Starknet**](https://www.starknet.io/) is a **Layer 2 ZK-Rollup** built on Ethereum, using **STARK proofs** to enable scalable, low-cost, and secure smart contract execution. Developed by [**StarkWare**](https://www.starkware.co/), it executes transactions off-chain via a **sequencer**, which produces a proof verified on Ethereum for finality. Starknet contracts are written in [**Cairo**](https://www.cairo-lang.org/), a language purpose-built for STARK-based computation. The network operates through custom full nodes like [**Pathfinder**](https://github.com/eqlabs/pathfinder), a Rust-based client that supports RPC access, state diffs, and detailed Cairo VM execution traces.

To support high-performance data indexing and streaming, Starknet integrates with [**Firehose**](https://github.com/streamingfast/firehose-starknet), a gRPC-based blockchain data pipeline developed by StreamingFast. Firehose captures deterministic block data, Cairo VM steps, and L2-specific state updates, enabling real-time analytics and verifiable indexing with frameworks like **Substreams** or [**The Graph**](https://thegraph.com/). This architecture empowers developers to build scalable, trustless data services on top of Starknet’s advanced ZK-rollup infrastructure.\
\
You'll find node guides below for:

* Pathfinder
* Juno
* Madara

With additional instructions for

* Substreams to Graph Node
* JSON RPC to Firestark
