---
label: Python
title: Warthog SDK for Python
---

# SDK for Python

Warthog provides a [`warthog_py`](https://github.com/warthog-network/warthog_py) library for the Python ecosystem (Django, FastAPI, requests, etc.). It is the Python port of the TypeScript [`warthog-ts`](typescript.md) library and supports the full set of 7 DeFi transaction types.

For a low-level reference of how to sign and submit transactions in Python without this library, see [Wallet Integration](../integration/wallets.md) — the conventions used here (`coincurve` for RFC6979 signing with low-s normalization, `pycryptodome` for RIPEMD-160, etc.) match that guide.

## Installation

The package is **not yet published to PyPI**. Install it directly from the GitHub repository using [uv](https://docs.astral.sh/uv/):

```bash
uv add "warthog_py @ git+https://github.com/warthog-network/warthog_py.git"
```

Or add it to your `pyproject.toml`:

```toml
[project]
dependencies = [
    "warthog_py @ git+https://github.com/warthog-network/warthog_py.git",
]
```

### Pinning to a branch

```toml
dependencies = [
    "warthog_py @ git+https://github.com/warthog-network/warthog_py.git@master",
]
```

### Requirements

- Python `>= 3.10`
- The runtime dependencies (`coincurve`, `mnemonic`, `bip32`, `pycryptodome`, `requests`) are installed automatically by `uv` along with the package.

## Features

- Generate wallets and HD accounts, sign transactions
- Built-in HTTP client for node communication
- All 7 signed DeFi transaction types: `wartTransfer`, `tokenTransfer`, `assetCreation`, `cancelation`, `liquidityDeposit`, `liquidityWithdrawal`, `limitSwap`
- Idiomatic Python API: `name(...)` raises typed exceptions, `try_name(...)` returns `None`
- Signing uses `coincurve` (libsecp256k1) `sign_recoverable` with BIP-62 low-s normalization (matches the convention in the Python integration guide)

## Quick Start

```python
from warthog_py import (
    Account,
    Address,
    Funds,
    Liquidity,
    NonceId,
    Price,
    RoundedFee,
    TokenDecimals,
    Wart,
    WarthogDeFiApi,
)

# 1. Load or generate your account
account = Account.from_private_key_hex("your-private-key-hex")

# 2. Prepare the recipient
recipient = Address.from_hex("0000000000000000000000000000000000000000de47c9b2")

# 3. Connect to a core/defi node. This client reads the V2 /chain/head format.
api = WarthogDeFiApi("http://127.0.0.1:3100")

# 4. Fetch the chain head and build a transaction context
ctx = api.create_transaction_context(RoundedFee.min(), NonceId.random())

# 5. Build and sign a WART transfer
tx = ctx.transfer_wart(account, recipient, Wart.from_e8(100_000_000))

# 6. Submit
result = api.submit_transaction(tx)
print(result["txHash"])
```

## The nonce

The nonce is a replay tag, not a counter. A transaction is identified by its
`(account, pin height, nonce)` triple, and the only check a node makes is
that it has not seen that triple before: a repeat is rejected with `ENONCE`,
unless the new transaction pays a higher fee, in which case it replaces the
old one.

The nonce does not have to increase, and no node reports an account's next
nonce, so there is nothing to fetch and nothing to keep track of. Use the
random one:

```python
from warthog_py import NonceId

nonce = NonceId.random()
```

## Try/convention

Every factory ships in two flavors following standard Python practice:

- **`name(...)`** — raises a typed exception (e.g. `InvalidAddressError`, `InvalidNonceError`) on invalid input.
- **`try_name(...)`** — returns `None` on invalid input instead of raising.

```python
Address.from_hex("0000000000000000000000000000000000000000de47c9b2")  # raises on invalid
Address.try_from_hex("0000000000000000000000000000000000000000de47c9b2")  # None on invalid
Address.validate("0000000000000000000000000000000000000000de47c9b2")  # bool
```

All custom exceptions inherit from `warthog_py.errors.WarthogPyError`.
