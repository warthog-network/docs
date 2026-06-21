# Documentation gaps

Internal planning note — **not published** (Retype skips underscore-prefixed paths). Lists topics that are referenced by the project, exist in the source code, or are part of the HUB-level feature set, but do **not** yet have a published page under `docs/`.

Use this as a backlog for documentation work. When you add a new page, remove the line (or check it off). Items here are cross-referenced with the hub source-of-truth (see `../HUB.md`) — if a fact changes there, the gap (or the page that fills it) may need to change too.

## 1. Wallets

| Gap | Source of truth | Notes |
|-----|-----------------|-------|
| **Wartlock** (mainnet desktop, Electron/Bun) | `Wartlock/` repo | Mentioned in HUB.md and `links.md`-adjacent sites but no user-facing setup guide. Should include install, restore, send/receive. |
| **mobile-wallet** (React Native/Expo, testnet) | `mobile-wallet/` repo | Listed in `links.md` socials page indirectly; no dedicated guide. Document testnet-only status. |
| **wart-wallet** (Python GTK legacy, mainnet) | `wart-wallet/` repo | Not in `links.md`; HUB.md marks it as legacy standalone. Either document it briefly or remove from inventory. |
| **Wallet comparison matrix** | HUB.md `## Wallet Network Compatibility` | One page showing: name, network (mainnet/testnet), platform, signing method (warthog-ts or custom), repo, build state. Currently each wallet is documented in isolation. |
| **Wallet security model** (key storage, signing flow) | `warthog-ts/`, `browser-wallet/` | No page explains the local-signing architecture, mnemonic → seed → HD path flow, or how a non-custodial wallet talks to a node. |

## 2. Network state and constants

| Gap | Source of truth | Notes |
|-----|-----------------|-------|
| **Mainnet vs testnet (DeFi) overview** | HUB.md `## Project State` | `links.md` and individual pages mention testnet DeFi in passing, but there's no top-level "Networks" page. Users don't know `wartTransfer` is the only mainnet tx type. |
| **Key constants reference page** | HUB.md + `core/defi/meson.build` | One page (or sidebar) listing chain ID `0x539` (1337), block version 4, Verus v2.2, block time 20s, initial reward 3 WART, decimals 8, HD path `m/44'/2070'/0'/0/{index}`. Currently scattered. |
| **Emission schedule / reward halving** | `whitepaper/files/emission_scheme.csv` | Referenced in whitepaper; no docs version. |
| **Block structure** | `core/defi` headers, whitepaper | Whitepaper has it; docs has nothing. |
| **Transaction format reference** | `core/defi/src/shared/src/communication/create_transaction.hpp` | Common prefix `pinHash(32) + pinHeight(4) + nonceId(4) + reserved(3) + compactFee(2) = 45 bytes` plus the 7 signed-tx types. Currently only via the C++ source comment; no docs page. |
| **Address format** | `core/defi` | 20-byte address, 48-hex + 4-byte checksum = 52-hex base58/bech32 encoding. Not in docs. |
| **Fee model** | `core/defi/src/shared/src/defi/uint64/pool.hpp` (`feeE4 = 10` = 0.10%) and order-book fee rules | Currently only in FBM paper. |

## 3. Fair Batch Matching / DeFi

| Gap | Source of truth | Notes |
|-----|-----------------|-------|
| **End-user "How to use DeFi"** | `core/defi` + FBM paper | `unique-features/hard-coded-defi/fair-batch-matching.md` is the protocol-theory doc; there's no walkthrough of: create asset, create pool, deposit liquidity, place limit order, cancel order. |
| **Per-transaction-type walkthroughs** | `core/defi/src/shared/src/communication/create_transaction.hpp` | 7 DeFi tx types (`tokenTransfer`, `assetCreation`, `limitSwap`, `liquidityDeposit`, `liquidityWithdrawal`, `cancelation`, plus `wartTransfer`). Each should have a "what it does, fields, signing" page. |
| **Pool trading walkthrough** | `core/defi/src/shared/src/defi/uint64/pool.hpp` | No doc on the actual AMM process: deposit, swap, withdraw, fee accrual, k-invariant, slippage. |
| **Order book walkthrough** | `core/defi` matcher | No doc on placing/cancelling limit orders, what "no remaining pair" looks like in practice, partial fills, price-level aggregation. |
| **Counter-order paradox in user-friendly terms** | FBM paper + `unique-features/hard-coded-defi/fair-batch-matching.md` | Theory exists; no worked example with numbers. |
| **DeFi address/asset creation** | `core/defi` | Walkthrough of `assetCreation` tx: name length (5 bytes), ticker, hash derivation. |
| **DeFi fees and slippage** | `core/defi/src/shared/src/defi/uint64/pool.hpp` | Pool fee (0.10%) is hard-coded but never explained in docs. |

## 4. Mining

| Gap | Source of truth | Notes |
|-----|-----------------|-------|
| **Pool mining user guide** | `developers/integration/pools/pools.md` and `stratum.md` | The integration docs explain how a pool talks to a node, but a user wanting to mine to a pool (point a miner at a pool URL, configure worker, monitor) has no walkthrough. |
| **Detailed bzminer setup** | `developers/integration/miners.md` | Quickstart uses bzminer; advanced options (multi-GPU tuning, overclocking, OC profiles) aren't covered. |
| **janusminer setup** | `developers/integration/miners.md` | Listed as open-source, discontinued; no current setup guide. |
| **Solo vs pool mining comparison** | docs | The `node/node.md` page mentions solo mining briefly; no comparison page for when to choose which. |
| **Reward halving schedule** | whitepaper / source | See "Emission schedule" above. |
| **More i18n quickstarts** | `quickstart_fr.md` exists | Spanish, German, Chinese, Turkish, Russian quickstarts. The community has YouTube tutorials in those languages (per `links.md`). |

## 5. Node operation

| Gap | Source of truth | Notes |
|-----|-----------------|-------|
| **Bridge node setup** | `_drafts/bridge-node-setup.md` | Drafted but not promoted. Should describe a node that bridges TCP and WebSocket transports. |
| **Browser node setup** | `_drafts/WebRTC.md` + `unique-features/browser-nodes/webrtc-protocol.md` + `unique-features/browser-nodes/Introduction.md` | Protocol and intro docs exist; no "How to run a browser node" end-user guide. Status per HUB.md: on hold. |
| **Mining node (with stratum) setup** | `node/node.md` | A dedicated page combining node setup + stratum config, with a worked example, would help. |
| **Public RPC node setup** | `developers/api/rest.md` has a callout; HUB.md `### Public RPC Node` lists the CLI flag | The flag is mentioned; a dedicated "How to run a public RPC node" page (with `--enable-public`, security caveats, port forwarding, what's exposed vs hidden) is missing. |
| **Node config file** | `core/defi` | There's no docs page on the node config file format (key/value overrides, defaults, examples). |
| **Backup and restore** | `core/defi` | A wallet/node backup walkthrough (seed phrase, datadir, chainstate) is missing. |
| **Update / upgrade procedure** | `core/defi` Docker / GitHub releases | No "how to upgrade" guide; the releases page lists binaries but doesn't tell users how to migrate. |
| **Branch / fork guidance** | HUB.md `core/master` vs `core/defi` | A short page: "If you want DeFi, run `defi`; if you want the stable mainnet chain, run `master`." |

## 6. API

| Gap | Source of truth | Notes |
|-----|-----------------|-------|
| **WebSocket protocol details** | `developers/api/websocket.md` | The page exists but is a stub. Needs: subscription model, event names, message format, ping/pong, reconnection guidance. |
| **Asset-related endpoints** | `core/defi/src/node/api/hook_endpoints.hxx` | `assetCreate`, `assetTransfer`, `pool*`, `order*` — none individually documented. |
| **Pool-related endpoints** | same | `poolCreate`, `poolDeposit`, `poolSwap`, `poolWithdraw` — no docs. |
| **Order-book endpoints** | same | `limitSwapCreate`, `limitSwapCancel` — no docs. |
| **Public RPC vs private RPC** | `developers/api/rest.md` and HUB.md | Currently a callout in `rest.md`; could be its own page that lists which endpoints are public, which are private, and why. |
| **Error code reference** | `core/defi` HTTP endpoint | EIP-1193-style codes (`4001`, `4100`, `-32603`, `-32601`) are listed for browser-wallet; no general API error-code reference. |
| **Rate limits / DoS protection** | `core/defi/src/node/api/http/endpoint.cpp` | No docs on what limits exist (per-IP, per-method, response on breach). |

## 7. SDK / libraries

| Gap | Source of truth | Notes |
|-----|-----------------|-------|
| **Cookbook / examples** | `warthog-ts/examples/` (per HUB.md) | The `examples/` folder exists in the SDK; no docs page linking the recipes (send transaction, sign message, listen to events, integrate with React). |
| **Other language bindings** | `warthog-ts/` | TypeScript is the only documented binding. If there are others (e.g., community Rust/Python ports), they're not listed. |
| **SDK versioning** | HUB.md `### warthog-ts Versioning` | A page explaining the published-on-npm cadence, semver policy, and "is the latest SDK compatible with the latest node" — currently known to drift. |

## 8. Concepts and reference

| Gap | Source of truth | Notes |
|-----|-----------------|-------|
| **Glossary** | HUB.md + whitepaper + FBM | No central A-Z of terms: Janushash, PoBW, FBM, APU mining, Nash equilibrium, signed vs implicit, k-invariant, etc. |
| **FAQ** | `website/src/pages/faqs.astro` (in website repo) | The website has a FAQ; docs has no equivalent. Either link to it or maintain a docs-side one. |
| **Architecture overview** | HUB.md `## Blockchain` | One diagrammed page: node ↔ mempool ↔ chain ↔ consensus ↔ API ↔ wallets ↔ miners. |
| **Troubleshooting** | issues on GitHub | No "common errors" page (e.g., "stratum connection refused", "out of sync", "RPC port 3000 vs 3001", "OpenCL missing"). |
| **Security model** | `core/defi`, `warthog-ts` | No page on what a node does and doesn't validate, threat model for public RPC, what's trusted vs untrusted. |
| **Mempool / transaction lifecycle** | `developers/api/rest/mempool-transactions.md` | Page exists; verify it covers propagation, ordering, and reorg behaviour. |

## 9. Project / community

| Gap | Source of truth | Notes |
|-----|-----------------|-------|
| **Contributing guide** | GitHub | No `CONTRIBUTING.md` linked from docs. Should cover: PR process, code style per sub-repo, where to file issues, how to propose protocol changes. |
| **Changelog / version history** | `_version_history.md` (in `_` prefix, not in nav) | The file exists but isn't published. Either promote it to a real changelog page or keep a per-release summary on `readme.md`. |
| **License** | per-repo `LICENSE` | No top-level docs page pointing to per-repo licenses (GPL-3 for core, MIT for warthog-ts, etc.). |
| **Team / contributors** | `whitepaper/`, GitHub | Whitepaper credits Pumbaa, Rafiki, CoinFuMasterShifu. No equivalent page on the docs site. |
| **Press kit / brand usage** | `../brand-kit/` | `links.md` has a single link; no guidance on how to use the logo, brand colors, etc. for press / community content. |

## 10. Cross-cutting

| Gap | Notes |
|-----|-------|
| **Docs versioning per node version** | `rest.md` says "API for Warthog node version v0.10.16" but there's no version selector. Past versions are lost. |
| **"Known inconsistencies" page** | HUB.md `## Known Inconsistencies` lists 5 divergent node URL lists, version drift, Docker tag collision. A docs page that surfaces these to users (with mitigations) would prevent confusion. |
| **More i18n** | Only the mining quickstart is translated (to French). The rest of the docs is English-only. |
| **Accessibility audit** | No docs-side a11y review (alt text, color contrast, keyboard nav). |
| **Diagram-heavy pages without images** | Several FBM/Janushash theory pages have heavy ASCII or no diagrams. A image-first pass would help. |

## 11. Source-code features that have no docs

A short list of features that exist in `core/defi` and are referenced in the whitepaper or HUB.md, but for which no `docs/` page exists:

- **C++23 codebase** — `core/defi/meson.build`; the docs say nothing about implementation language.
- **Signed vs Implicit transactions** — terminology is used in the whitepaper; docs should adopt it.
- **Pool k-invariant and price-impact formula** — in `core/defi/src/shared/src/defi/uint64/pool.hpp`; no user-facing math doc.
- **Recoverable signature (65 bytes) / CompactUInt (2 bytes)** — protocol primitives; no byte-layout reference.
- **`payoutid` / `payoutid_drain` mechanics** — implicit transactions; no docs.
- **Bridge / WebRTC transport** — see §5 above.
- **Browser node OPFS persistence** — mentioned in `unique-features/browser-nodes/Introduction.md` but no operational doc.

## 12. Verified coverage (no gap)

For completeness, the following are **already documented** in the published site (per the file tree at AGENTS.md authorship time):

- Mining quickstart (EN, FR), calculator, OpenCL install.
- Node: setup, compiling, Docker, systemd, defi-testnet branch, GCC-from-source.
- Public-data: index + 4 sub-pages (addresses, assets, hashrates, nodes).
- Wallets: browser-extension, CLI, wart-dapp.
- Browser-wallet EIP-1193 integration.
- REST API + REST sub-pages (transactions, create-transaction, mempool-transactions), WebSocket stub.
- Integration: miners, pools, stratum, wallets.
- Libraries: TypeScript.
- Unique features: FBM (canonical), hard-coded DeFi, Janushash + 4 analysis sub-pages, browser-nodes intro and WebRTC protocol.

## How to use this file

1. Pick an item.
2. Cross-reference the listed source of truth (likely HUB.md or a sub-repo) before writing.
3. Write the page; remove the line (or convert to `[x]`) here.
4. If the page belongs in a folder that has an `index.yml`, add it for sidebar ordering.
5. Update the matching section in `AGENTS.md` (Folder layout) so future agents see the new file.
