# DeFi Testnet

The defi branch introduces new DeFi functionality to Warthog — including native token creation, a decentralized exchange (DEX) with limit orders, and liquidity pools. The testnet exists to validate these features before mainnet activation.

This guide covers how to run a testnet node and use the new DeFi features.

Here is how to run a defi testnet node:
We need some people to set up a testnet for the defi branch. THis is the branch we have been working on for a very long time. It is now time to start the testnet phase and to observe how the new node behaves. During this process, we hope to observe and fix final bugs in the node but overall, the code seems to work well and stable. Yet more testing is necessary especially concerning the new DeFi functionality. It may happen that the testnet will be restarted if critical bugs are found that require a fresh chain. 

## Compatibility and DeFi upgrade on mainnet
The defi branch code is carefully written in a way to be compatible with the current main net. This compatibility mode is the default starting mode. The idea is, once testnet was stable and the new features shall come to the main node, to specify an **upgrade height** where the new DeFi features become active. For now we have set this height to a very large value, effectively disabling the feature upgrade height. This means that the new node shall essentially behave like the current mainnet node except for the internals which are very different now due to a completely changed database table architecture. However from a user perspective, starting the `defi` branch node normally without the testnet flats, it should behave like the mainnet node. This compatibility will be tested after the testnet.

## Testnet flag
When the defi branch node is started with the `--testnet` flag, the upgrade height is ignored and the DeFi features are available from the beginning. 

## Mainnet Upgrade Path

When the testnet has been battle-tested and the new features are ready for production, an **upgrade height** is set on mainnet. Until that height is reached, the defi branch node operates in compatibility mode — it behaves identically to the current mainnet node, so existing tools and workflows continue to work unchanged. At the upgrade height, DeFi features activate automatically. The new database architecture is invisible to users in this mode.

## Starting a Testnet Node

Run the defi branch node with the `--testnet` flag:

```bash
./wart-node --testnet
```

This ignores the upgrade height and makes DeFi features available from block 1. 

## Using DeFi Features

The testnet includes a new TUI wallet for DeFi interactions. Currently supported features:

- **Token creation** — create new tokens with a given name and total supply
- **Liquidity pools** — deposit base asset and WART to receive LP shares; redeem LP shares to withdraw liquidity
- **Limit orders** — place buy or sell limit orders for any token against WART
- **Token transfers** — transfer WART or any other token

## Blockchain Explorer

A new blockchain explorer for the testnet is available at [testnet.warthog.network](https://testnet.warthog.network).
These features need a  new blockchain explorer which is actively worked on but is already in a state which could be useful for starting the testnet. 

## TUI Wallet

The new defi features can be accessed with a new TUI wallet which is compiled automatically together with the [`defi` branch node in the Warthog Core Repository](https://github.com/warthog-network/core/tree/defi). This wallet just an early stage proof of concept and is not hardened for security. It can be used to create new tokens, interact with pools and place buy and sell orders for non-wart tokens. 

Relative to the build directory the TUI wallet is located at `./src/tui_wallet`.
Wallet Assets Requests                                                                                                                                                               
```
──────╴──────╶────────────────────────────────────────────────────────────────                                                                                                       
╭Select Asset────────────────────────────────────────────────────────────────╮                                                                                                       
│Name:                                                                       │                                                                                                       
│Hash:                                                                       │                                                                                                       
│WART 0000000000000000000000000000000000000000000000000000000000000000       │                                                                                                       
│w 49ce963b1b69cb798b1be9a646b60019b35164684752f93cb9b181c9825ac7ba          │                                                                                                       
│ASDF bf31bdb0e4bb051698deabaadc63f64f83fe9005dbd53d2869808dcbc539c636       │                                                                                                       
│df c6f70c735a319f8790c080b586db4dbabc1b53d1f38c47d81554db5b1c1edac2         │                                                                                                       
╰────────────────────────────────────────────────────────────────────────────╯                                                                                                       
╭Selected Asset──────────────────────────────────────────────────────────────╮                                                                                                       
│Name       w                                                                │                                                                                                       
│Hash       49ce963b1b69cb798b1be9a646b60019b35164684752f93cb9b181c9825ac7ba │                                                                                                       
│Decimals   5                                                                │                                                                                                       
│           ┌────┐                                                           │                                                                                                       
│           │Fork│                                                           │                                                                                                       
│           └────┘                                                           │                                                                                                       
│Balance    Total : 999.49694                                                │                                                                                                       
│           Locked:   0                                                      │                                                                                                       
│           Free  : 999.49694                                                │                                                                                                       
│           ┌────────┐                                                       │                                                                                                       
│           │Transfer│   Buy      Sell                                       │                                                                                                       
│           └────────┘                                                       │                                                                                                       
│Liquidity  Total : 0.03143859                                               │                                                                                                       
│           Locked: 0                                                        │                                                                                                       
│           Free  : 0.03143859                                               │                                                                                                       
│           ┌────────┐                                                       │                                                                                                       
│           │Transfer│ Deposit  Withdraw                                     │                                                                                                       
│           └────────┘                                                       │                                                                                                       
╰────────────────────────────────────────────────────────────────────────────╯
```

## Disclaimer

The testnet may be restarted if critical bugs are found. In that case all testnet coins are reset to 0 and all testnet-created tokens are cleared. **Do not assign any value to testnet coins or testnet-created tokens.**
