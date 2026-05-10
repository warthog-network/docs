---
label: Hashrates
---

# Submitting Hashrates

Hashrate data helps miners estimate their expected Janushash performance when choosing GPU and CPU configurations.

## Adding Your Hashrate Data

1. Fork the [public-data repository](https://github.com/warthog-network/public-data)
2. Edit the relevant CSV file in `data/`:
   - `sha256t-hashrates.csv` - for GPU hashrates
   - `verushash2_2-hashrates.csv` - for CPU hashrates
3. Add a new line with the format: `manufacturer,model,hashrate mh/s`
4. Submit a pull request to the `master` branch

### CSV Format

| Field | Description |
|-------|-------------|
| `manufacturer` | Hardware manufacturer (e.g., "AMD", "Nvidia", "Intel") |
| `model` | Model name (e.g., "RTX 4090", "Ryzen 9 7950X") |
| `hashrate mh/s` | Hashrate in megahashes per second |

### Example Entries

**GPU (sha256t):**
```
Nvidia,RTX 4090,5000
AMD,RX 7800 XT,2200
```

**CPU (verushash2_2):**
```
AMD,Ryzen 9 7950X,50
Intel,Xeon E5-2696 (2x),60.00
```

## API Endpoints

Once your data is merged and deployed:

| Endpoint | Description |
|----------|-------------|
| [https://data.warthog.network/sha256t-hashrates.json](https://data.warthog.network/sha256t-hashrates.json) | GPU hashrates |
| [https://data.warthog.network/verushash2_2-hashrates.json](https://data.warthog.network/verushash2_2-hashrates.json) | CPU hashrates |

## Need Help?

Join the Warthog community on [Discord](https://discord.com/invite/QMDV8bGTdQ) for assistance.
