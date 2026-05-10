# Adding an Address Tag

Address tags help identify known WART addresses such as exchange wallets, pool payouts, and donation addresses.

## Adding Your Address Tag

1. Fork the [public-data repository](https://github.com/warthog-network/public-data)
2. Edit `data/addresses.csv`
3. Add a new line with the format: `address,tag,comment`
4. Submit a pull request to the `master` branch

### CSV Format

| Field | Description |
|-------|-------------|
| `address` | Full WART address (64 character hex string) |
| `tag` | Short label identifying the address (e.g., "Herominers", "Donations") |
| `comment` | Optional comment (can be empty) |

### Example Entry

```
95ae6efb2f4fe5e4fd3a5b21df7f755f878383610505fe64,Herominers,
```

## API Endpoint

Once your tag is merged and deployed:

| Endpoint | Description |
|----------|-------------|
| [https://data.warthog.network/addresses.json](https://data.warthog.network/addresses.json) | WART address tags |

## Discussion

Join the Warthog community on [Discord](https://discord.com/invite/QMDV8bGTdQ) for assistance.
