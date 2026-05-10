# Registering Asset Metadata

Warthog stores asset metadata off-chain in the public-data repository. This allows anyone to register information about assets on the Warthog chain.

## Prerequisites

- You must have an **asset hash** from the Warthog blockchain
- Contact the team on [Discord](https://discord.com/invite/QMDV8bGTdQ) to have your asset hash registered

## Adding Your Asset

1. Fork the [public-data repository](https://github.com/warthog-network/public-data)
2. Create a new directory: `data/assets/{your_asset_hash}/`
3. Create `info.json` with the following structure:
4. Optionally add `image.png` (valid PNG file)
5. Submit a pull request to the `master` branch

### Directory Structure

```
data/assets/{asset_hash}/
├── info.json      # Required
└── image.png      # Optional
```

### info.json Schema

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
    "description": "WART is the native token in the Warthog ecosystem...",
    "url": "https://warthog.network"
}
```

## API Endpoints

Once your asset is merged and deployed:

| Endpoint | Description |
|----------|-------------|
| [https://data.warthog.network/assets.json](https://data.warthog.network/assets.json) | Asset list for autocompletion |
| [https://data.warthog.network/assets/\{hash\}/info.json](https://data.warthog.network/assets/0000000000000000000000000000000000000000000000000000000000000000/info.json) | Asset metadata |
| [https://data.warthog.network/assets/\{hash\}/image.png](https://data.warthog.network/assets/0000000000000000000000000000000000000000000000000000000000000000/image.png) | Asset logo (optional) |

## Need Help?

Join the Warthog community on [Discord](https://discord.com/invite/QMDV8bGTdQ) for assistance.
