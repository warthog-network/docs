---
title: Docker
label: Docker
order: 2
---

# How to run a warthog node with docker.

## Basic way

```sh
docker run zzjulien/warthog_node:latest
```

## Modify your node

For example, you can add this type of arguments to run a node with stratum enabled

```sh
docker run --expose 3456 zzjulien/warthog_node:latest --stratum=0.0.0.0:3456
```

For full miner configuration (BzMiner flags, pool vs solo), see [Solo mining quickstart](../mining/quickstart).

## Run a (RPC) public node

```sh
docker run --expose 3001 zzjulien/warthog_node:latest --enable-public
```

`--enable-public` exposes a filtered subset of the API on port 3001. Critical admin endpoints (e.g., `POST /chain/append`, `/debug/rollback`, peer admin) are not exposed. The default RPC port (3000) is full-access and should never be exposed to the internet — use a firewall to restrict access to port 3000.

For a persistent setup with auto-restart, a proper systemd unit file, and full security rationale, see [How to start a public node](./systemd#how-to-start-a-public-node).

## Run a defi testnet node

Add `--testnet` to enable the defi testnet chain (token creation, DEX, pools, FBM). The image is the same; only the chain and feature flags differ:

```sh
docker run zzjulien/warthog_node:latest --testnet
```

See [DeFi Testnet](./defi-testnet) for details on what the testnet enables and how to interact with it.

!!!info Info
We advise you not to use the latest image, but rather to choose a stable version (latest release), currently zzjulien/warthog_node:0.10.16
!!!

!!!info Info
Warthog image is whitelisted on [runonFlux](https://runonflux.io/)
!!!
