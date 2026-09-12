---
title: Using a Node
label: Node
order: 1
---
# Using a Node

## Overview
Warthog's node is the program that talks to other nodes on the peer-to-peer network. Once started, the node synchronises the blockchain from peers and keeps up with new blocks which are mined roughly every 20s.

For interaction with the Warthog network as required by wallets, explorers and developers, nodes provide two forms of API endpoints:
- The **local JSON API endpoint** provides for full inspection and control and therefore should only be used in safe environments, especially it should not be exposed to the internet.
- The **public JSON API endpoint** provides a safe restricted subset of API methods for public exposure to the internet used by wallets and explorers.

Nodes also support mining based on HTTP API or Stratum protocol without extra tools.

### Networks

There are two networks running different code:

- **Mainnet**: current production network. The node binary from the [`master`](https://github.com/warthog-network/core/tree/master) branch serves this. Only transfer of coins is available, no DeFi features.
- **DeFi Testnet**: runs the [`defi`](https://github.com/warthog-network/core/tree/defi) branch code. Adds token creation, the DEX, limit orders, liquidity pools, and [Fair Batch Matching](../../unique-features/hard-coded-defi/fair-batch-matching). Start with `--testnet`.

For the testnet setup, see [DeFi Testnet](./defi-testnet).

## Starting a node

There are three ways to install the Warthog node:
1. [Download pre-compiled executables](https://github.com/warthog-network/Warthog/releases) are the fastest way to get started. Just download and run without any dependencies.
2. [Compile from source](./compiling) when you want a custom build, the latest unreleased code, or to contribute to the source.
3. [Use Docker](./docker) for easy server deployments.

Assuming you are on Linux and have either downloaded or compiled the Warthog node executable `wart-node-linux` you can start it by typing

```bash
./wart-node-linux
```

If you are connected to the internet, the node will start syncing blocks with the network.

![](/img/get-started/09-node.png)

Once synchronization has finished the node will print a line similar to this:
```
Synced in 3min 45s. (height 987654).
```


## Running a public node

A **public node** is a node that exposes its public JSON API endpoint to the internet. This helps users conveniently issue requests against the network as needed by explorers and wallets. We provide a list of [public mainnet nodes](https://data.warthog.network/legacy-nodes.json) and [public testnet nodes](https://data.warthog.network/defi-nodes.json).

**Enable the public API**

Start the node with the `--enable-public` flag (shorthand for `--publicrpc=0.0.0.0:3001`):

```bash
./wart-node-linux --enable-public
```

This exposes a **filtered subset** of the API on port **3001**. Critical admin endpoints (`POST /chain/append`, peer admin, debug tools) are not exposed. The default private RPC on port 3000 is full-access and **must never be exposed to the internet**. Bind it to localhost (`--rpc=127.0.0.1:3000`) or firewall it.

For a persistent setup with auto-restart and a proper unit file, see [Systemd](./systemd). For the Docker equivalent, see [Docker](./docker).

**Security checklist**

- Bind port 3000 to `127.0.0.1` (or block it at the firewall). Only the port for the public API endpoint should be public.
- Keep the node version up to date.
- If you also mine, expose port 3456 only to your miner (not the open internet).

**Register your node**

The community is welcome to host more public nodes for improved network resilience. To support us in this way you need a **static public IP** or a stable DNS name and an opened inbound port to make the node's public API accessible for everyone.
Once the node is running and reachable, [submit it to the public node list](../public-data/nodes) or contact us in the [Warthog Discord](https://discord.com/invite/QMDV8bGTdQ) so it appears at `data.warthog.network`. There are two lists:

- [`legacy-nodes.json`](https://data.warthog.network/legacy-nodes.json) for mainnet (master branch).
- [`defi-nodes.json`](https://data.warthog.network/defi-nodes.json) for testnet (defi branch).

A public node that is not registered is reachable but may not be discoverable to users.

## Run your own node for solo mining

[BzMiner](https://www.bzminer.com/) supports both Warthog's [REST API](/developers/api/rest.md) and the [Stratum protocol](/developers/integration/pools/stratum.md). To use stratum, users must explicitly start the node with the `--stratum` flag and specify the bind IP and port:

```bash
./wart-node-linux --stratum=0.0.0.0:3456
```

The same port must be configured on your miner (BzMiner by default). The miner then connects to the node, requests work, and submits shares. For a quick start on mining, see [Solo mining quickstart](../mining/quickstart).

## Troubleshooting

If something does not work as expected:

- **Port already in use**: another process is bound to 3000 / 3001 / 3456. Pick a different port with `--rpc=…`, `--publicrpc=…`, `--stratum=…`.
- **Node won't start**: check the log for `Cannot listen on <ip:port>`. Usually a missing `--enable-public` (if you wanted 3001) or a permission error on a low port.
- **Public node not reachable from outside**: confirm the firewall allows inbound TCP/3001 and that your cloud provider's security group also opens it.

Still stuck? Ask on the [Warthog Discord](https://discord.com/invite/QMDV8bGTdQ).
