---
label: Elixir
title: Warthog SDK for Elixir
---

# SDK for Elixir

Warthog provides an SDK for the Elixir/Erlang ecosystem called [`warthog_ex`](https://github.com/warthog-network/warthog_ex) intended for interaction with the Warthog ecosystem.

## Installation

The package is **not yet published to Hex.pm**. Install it directly from
the GitHub repository:

```elixir
# mix.exs
def deps do
  [
    {:warthog_ex, github: "warthog-network/warthog_ex"}
  ]
end
```

Then:

```bash
mix deps.get
mix deps.compile
```
A working C toolchain may be required (the `ex_secp256k1` dependency ships prebuilt NIFs for common platforms; otherwise it builds from source)

## Features

- Generate wallets and HD accounts, sign transactions
- Built-in HTTP client for node communication
- All 7 signed DeFi transaction types: `wartTransfer`, `tokenTransfer`, `assetCreation`, `cancelation`, `liquidityDeposit`, `liquidityWithdrawal`, `limitSwap`
- Idiomatic Elixir API: `{:ok, value} | :error` returns, `name!/1` bang variants that raise on invalid input
- Battle-tested crypto via [`ex_secp256k1`](https://hex.pm/packages/ex_secp256k1) (Rust NIF backed by libsecp256k1) and BIP-39/BIP-32 via [`cryptopunk`](https://hex.pm/packages/cryptopunk)

## Quick Start

```elixir
alias WarthogEx.{
  Account,
  Address,
  Funds,
  Liquidity,
  NonceId,
  Price,
  RoundedFee,
  TokenDecimals,
  TransactionContext,
  Wart,
  WarthogApi
}

# 1. Load or generate your account
account = Account.from_private_key_hex!("your-private-key-hex")

# 2. Prepare the recipient
{:ok, recipient} = Address.from_hex("0000000000000000000000000000000000000000de47c9b2")

# 3. Connect to a node
api = WarthogApi.new("http://127.0.0.1:3100")

# 4. Fetch the chain head and build a transaction context
{:ok, ctx} = WarthogApi.create_transaction_context(api, RoundedFee.min(), NonceId.random())

# 5. Build and sign a WART transfer
tx = TransactionContext.transfer_wart(ctx, account, recipient, Wart.from_e8!(100_000_000))

# 6. Submit
{:ok, %{txHash: hash}} = WarthogApi.submit_transaction(api, tx)
```
