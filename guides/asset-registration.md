---
label: Asset Registration
---

# Asset Registration

Warthog stores asset metadata off-chain in the [public-data repository](https://github.com/warthog-network/public-data). This allows anyone to register information about assets on the Warthog chain, including descriptive names, ticker symbols, descriptions, and logos.

## Prerequisites

- You must have an **asset hash** from the Warthog blockchain
- If you don't have an asset hash yet, you need to create an asset on-chain first
- Contact the team on [Discord](https://discord.com/invite/QMDV8bGTdQ) to have your asset hash registered in the public-data repository

## Directory Structure

Each asset is stored in a directory named after its hash:

```
data/assets/{asset_hash}/
├── info.json      # Required: asset metadata
└── image.png      # Optional: asset logo (must be valid PNG)
```

## info.json Schema

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Long name of the asset (e.g., "Warthog") |
| `ticker` | Yes | Short ticker symbol (max 5 characters, e.g., "WART") |
| `description` | Yes | Description of the asset |
| `url` | No | Website URL |

### Example info.json

```json
{
    "name": "Warthog",
    "ticker": "WART",
    "description": "WART is the native token in the Warthog ecosystem. Fees are paid in it and every asset is traded against WART.",
    "url": "https://warthog.network"
}
```

## Steps to Register Your Asset

1. Fork the [public-data repository](https://github.com/warthog-network/public-data)
2. Create a new directory: `data/assets/{your_asset_hash}/`
3. Create `info.json` with the required fields
4. Optionally add `image.png` (valid PNG file, recommended size 256x256px)
5. Submit a pull request to the `master` branch

## API Endpoints

Once your asset is merged and deployed, the following endpoints will be available:

| Endpoint | Description |
|----------|-------------|
| [https://data.warthog.network/assets.json](https://data.warthog.network/assets.json) | List of all assets (hash, name, ticker) for autocompletion |
| [https://data.warthog.network/assets/{hash}/info.json](https://data.warthog.network/assets/0000000000000000000000000000000000000000000000000000000000000000/info.json) | Individual asset metadata |
| [https://data.warthog.network/assets/{hash}/image.png](https://data.warthog.network/assets/0000000000000000000000000000000000000000000000000000000000000000/image.png) | Asset logo image (optional) |

### Example Response

**GET /assets.json**
```json
[
  {"hash": "0000000000000000000000000000000000000000000000000000000000000000", "name": "Warthog", "ticker": "WART"}
]
```

**GET /assets/{hash}/info.json**
```json
{
  "name": "Warthog",
  "ticker": "WART",
  "description": "WART is the native token in the Warthog ecosystem...",
  "url": "https://warthog.network"
}
```

## Need Help?

Join the Warthog community on [Discord](https://discord.com/invite/QMDV8bGTdQ) for assistance with asset registration.