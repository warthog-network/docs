---
title: Elixir
---

# Elixir Library for Warthog

Warthog provides an [`warthog_ex`](https://github.com/warthog-network/warthog_ex) library for the Elixir/Erlang ecosystem (Phoenix, OTP, Nerves, etc.). It is the Elixir port of the TypeScript [`warthog-ts`](typescript.md) library and supports the full set of 7 DeFi transaction types.

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

### Pinning to a tag or branch

For reproducible builds, pin to a tagged release:

```elixir
def deps do
  [
    {:warthog_ex, git: "https://github.com/warthog-network/warthog_ex.git", tag: "v0.1.0"}
  ]
end
```

Or follow a branch:

```elixir
def deps do
  [
    {:warthog_ex, git: "https://github.com/warthog-network/warthog_ex.git", branch: "main"}
  ]
end
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

## Bang convention

Every factory function ships in two flavors following standard Elixir
practice:

- **`name/1`** — returns `{:ok, t()} | :error`. Use when failures are expected.
- **`name!/1`** — raises `ArgumentError` on invalid input. Use when you trust the input and want a tighter call site.

```elixir
{:ok, wart} = Wart.parse("1.5")        # safe
Wart.parse!("1.5")                     # raises on invalid
Wart.parse("1.123456789") == :error    # too many decimals for WART (8)
```

## Differences from `warthog-ts`

- **Naming**: `Account.fromRandom()` → `Account.from_random/0`, `wart.roundedFee(true)` → `Wart.rounded_fee(wart, true)`.
- **Errors**: `null` → `:error`; successful values wrapped in `{:ok, value}` tuples.
- **Bang convention**: every factory ships as both `name/1` (returns `{:ok, t()} | :error`) and `name!/1` (raises `ArgumentError`).
- **Struct fields**: `account.address` and `account.privateKeyHex` are public struct fields, not methods.

See the [`warthog_ex` AGENTS.md](https://github.com/warthog-network/warthog_ex/blob/main/AGENTS.md) for the full API reference and the project's own `examples/transactions.exs` for end-to-end usage.

## When to use which SDK

- **`warthog-ts`** is the canonical reference SDK. Use it when you're building a TypeScript / JavaScript client (Node, browsers, React Native, deno). New protocol features land here first.
- **`warthog_ex`** is the Elixir port. Use it when you're building on the BEAM (Phoenix web apps, OTP services, embedded devices with Nerves, etc.). Same wire format, same 7 transaction types.

Both libraries read from the same `KNOWN_NODES` list and target the same testnet (`core/defi`) API. If you need parity between a TypeScript frontend and an Elixir backend (e.g., signed by the browser wallet, verified by an OTP service), the two libraries produce identical transaction bytes for the same inputs.

## See also

- [`warthog-ts` TypeScript library](typescript.md) — the canonical reference.
- [`core/defi` REST API](../api/rest/transactions.md) — the on-chain schema these libraries implement.
- [`warthog_ex` source repository](https://github.com/warthog-network/warthog_ex) — AGENTS.md, examples, and tests.
