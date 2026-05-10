# Adding a Public Node

Running a public node helps the Warthog network by providing infrastructure for users who cannot run their own node.

## Prerequisites

First, set up your node. For detailed instructions, see the [Node Setup Guide](../node/node.md).

Once your node is running and synced, you can make it publicly accessible by configuring it to listen on a public IP or through a reverse proxy.

## Adding Your Node to the Repository

1. Fork the [public-data repository](https://github.com/warthog-network/public-data)
2. Edit the relevant CSV file in `data/`:
   - `legacy-nodes.csv` - for legacy nodes
   - `defi-nodes.csv` - for DeFi nodes
3. Add a new line with the format: `url,name,comment`
4. Submit a pull request to the `master` branch

### CSV Format

| Field | Description |
|-------|-------------|
| `url` | Full URL of your node (e.g., `http://65.87.7.86:3001`) |
| `name` | Your name or identifier |
| `comment` | Optional comment (can be empty) |

### Example Entry

```
http://65.87.7.86:3001,pumbaa,
```

## API Endpoints

Once your node is merged and deployed:

| Endpoint | Description |
|----------|-------------|
| [https://data.warthog.network/legacy-nodes.json](https://data.warthog.network/legacy-nodes.json) | Legacy public nodes |
| [https://data.warthog.network/defi-nodes.json](https://data.warthog.network/defi-nodes.json) | DeFi public nodes |

## Need Help?

Join the Warthog community on [Discord](https://discord.com/invite/QMDV8bGTdQ) for assistance.
