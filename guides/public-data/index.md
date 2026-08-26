---
label: Public Data
---

# Public Data

The [public-data repository](https://github.com/warthog-network/public-data) hosts public data for the Warthog community. This data is exported to API endpoints on [data.warthog.network](https://data.warthog.network).

Everyone is welcome to contribute by running public nodes, adding address tags, submitting hashrate data, or registering assets.

## Available Data

| Data Type | Description | Contribute |
|-----------|-------------|------------|
| [Public Nodes](nodes.md) | Community-run nodes for wallet access | Run a node and add it to the list |
| [Address Tags](addresses.md) | Labels for known WART addresses | Tag addresses you encounter |
| [Hashrates](hashrates.md) | GPU/CPU mining performance data | Submit your hashrate |
| [Assets](assets.md) | Metadata for tokens on the Warthog chain | Publish via the [asset metadata form](https://testnet-assets.warthog.network/) |

## API Endpoints

| Endpoint | Description |
|----------|-------------|
| [https://data.warthog.network/legacy-nodes.json](https://data.warthog.network/legacy-nodes.json) | Legacy public nodes |
| [https://data.warthog.network/defi-nodes.json](https://data.warthog.network/defi-nodes.json) | DeFi public nodes |
| [https://data.warthog.network/addresses.json](https://data.warthog.network/addresses.json) | WART address tags |
| [https://data.warthog.network/sha256t-hashrates.json](https://data.warthog.network/sha256t-hashrates.json) | GPU hashrates |
| [https://data.warthog.network/verushash2_2-hashrates.json](https://data.warthog.network/verushash2_2-hashrates.json) | CPU hashrates |
| <https://testnet-assets.warthog.network/assets.json> | Asset catalog (list) |
| <https://testnet-assets.warthog.network/assets/\{hash\}/info.json> | Single asset metadata |
| <https://testnet-assets.warthog.network/assets/\{hash\}/logo.png> | Asset logo (250×250 PNG) |
| <https://testnet-assets.warthog.network/assets/\{hash\}/banner.png> | Asset banner (600×200 PNG) |

## Discussion

Join the Warthog community on [Discord](https://discord.com/invite/QMDV8bGTdQ) for assistance or to discuss contributions.