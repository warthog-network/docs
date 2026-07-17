# Warthog's Main Features

Warthog is a cryptocurrency built from scratch around a handful of decisions that set it apart from every other project in the space. The categories below are the same as on [warthog.network](https://warthog.network) under "What makes us unique", with the longer, deeper-dive versions of each.

## At a glance

| Category | What it means |
|---|---|
| [Fair Tokenomics](#fair-tokenomics) | Fair launch, 0% team allocation, 100% minable, hard cap <19M |
| [Original Code](#original-code) | Written from scratch, modern C&#43;&#43;23 toolchain, open source |
| [BlitzSync](#blitzsync) | Blocks addressed by integer branch, not hash |
| [Janushash](#janushash-proof-of-balanced-work) | First production PoBW: GPU × CPU hash product |
| [Native DeFi](#native-defi) | DeFi primitives embedded in the consensus layer |
| [Fair Batch Matching](#fair-batch-matching) | Sandwich-proof DEX matching, Nash equilibrium pricing |
| [Browser Nodes](#browser-nodes) | Full nodes that run in any modern browser |

---

## Fair Tokenomics

Every WART in existence originates from public mining. There is no pre-mine, no reserved team fund, no advisor allocations, and no vesting cliffs. The supply curve is exactly the schedule the mining algorithm produces — no tail emission, hard cap around 19 million coins, scarcer than Bitcoin.

- **Fair Launch** — Publicly announced on Bitcointalk with no preallocation, premine, unlocks, or vesting cliffs.
- **0% Team Allocation** — No reserved dev fund, dev fees, hidden allocations, or advisor carve-outs. Development is funded by enthusiasts, community, and donations.
- **100% Minable** — Every token is publicly mineable by the community. The hard cap of no more than 19 million coins to ever be mined makes Warthog scarcer than Bitcoin.

This is the foundation that the rest of the design assumes: if insiders ever held privileged supply, the technical guarantees below would still hold, but the social guarantee would be gone.

---

## Original Code

Warthog is not a fork of Bitcoin, Ethereum, or any other chain. The entire stack — consensus, networking, storage, DeFi primitives — was purpose-built from scratch. The codebase uses a modern C&#43;&#43;23 toolchain (clangd, Meson, Ninja) chosen for auditability and build speed, and the repository is open source under the **MIT License**. Contributions are welcome across C&#43;&#43;, Rust, TypeScript, and design.

- **Written from Scratch** — No forks. Purpose-built stack designed for browser-native miners and hard-coded DeFi.
- **Modern Codebase** — C&#43;&#43;23, clangd, Meson, and Ninja keep the stack auditable and fast. No additional system libraries needed to compile.
- **Open Source** — Permissively licensed under MIT. Welcoming contributors across C&#43;&#43;, Rust, TypeScript, and design.

The choices that fall out of "from scratch" are visible in every corner of the codebase: the sync protocol, the storage layer, the DeFi matcher, even the file naming.

---

## BlitzSync

Warthog's entire sync model and communication protocol is built around **chain descriptors** — integer numbers that nodes use to describe their branches. To specify a specific block to download from a peer, only the height and the chain descriptor previously published by that peer are sufficient. To the best of our knowledge, no other cryptocurrency uses this approach.

- **Efficient** — sync is by far more light-weight in RAM, CPU, and bandwidth because no hashes need to be looked up, indexed, or transmitted.
- **Fast** — block ranges are addressed by height ranges, enabling batch downloads without any list of hashes.
- **Intelligent** — nodes understand where their own chain diverges from peers' and cut unnecessary requests, with branch-aware misbehavior detection that improves banning strategy.

The implementation lives in the `header_download` module under `core/dev/src/node/eventloop/sync/header_download/` (and the equivalent `core/master/` / `core/defi/` paths).

---

## Janushash (Proof of Balanced Work)

Warthog's Janushash is the first production **Proof of Balanced Work (PoBW)** algorithm: it multiplies two independent hash functions — Sha256t (GPU) and Verushash v2.2 (CPU) — so efficient mining requires hardware capabilities for both. This makes pure GPU farms uncompetitive (their CPUs bottleneck) and pure CPU farms or botnets uncompetitive (no accelerated Sha256t). The result is a network where typical gaming PCs can hold their own against large operations, edging closer to Satoshi's original "one CPU, one vote" vision.

- **Repelling Farms** — the required balanced combination bottlenecks pure GPU farms, pure CPU farms, and botnets.
- **Favoring Gamers** — mining requirements were carefully picked to place consumer gaming PCs in the efficiency sweet spot.
- **Democratized Mining** — mining rewards consumer hardware, not whoever can throw the most ASICs at the network.

The full mathematical treatment, including ASIC-resistance arguments, the hash-product interpretation, and the public mining pool ecosystem, is on the [Janushash](janushash/Janushash.md) page. The original PoBW paper lives at [warthog.network/PoBW.pdf](https://warthog.network/PoBW.pdf) and on [GitHub](https://github.com/CoinFuMasterShifu/ProofOfBalancedWork/blob/main/PoBW.pdf).

---

## Native DeFi

Instead of implementing DeFi on top of smart contracts, Warthog embeds the DeFi code directly into the consensus layer to open the door to unprecedented features.

- **User Protection** — post-creation supply inflation, MEV extraction, and other rug-pull backdoors are impossible by design. Tokenomics are reported by Warthog itself, not by third-party meme-coin dashboards.
- **Unique Features** — implementing DeFi natively unlocks features the world has not seen before, such as Fair Batch Matching (FBM) or token forks where new tokens are created with balances cloned from existing tokens.
- **Hardened Safety** — smart-contract-based DeFi is attacked almost weekly. Without an intermediate smart-contract layer, implementation is much safer to write, easier to reason about, and has fewer corner cases.

### Status

Native DeFi is **deployed on the testnet** (`core/defi` branch). The mainnet (`core/master`) only has `wartTransfer`. See the [mainnet vs testnet](#status) table in `docs/AGENTS.md` for the feature split.

---

## Fair Batch Matching

Fair Batch Matching processes all swap orders and pool liquidity in a block **jointly**, assigning the same conversion price to every buy and every sell. There is no ordering dependency to exploit — sandwich attacks, front-running, and back-running become impossible by construction. The matching is unique (a Nash equilibrium) and the algorithm finds it in polynomial time.

For the full mathematical treatment, see the [Fair Batch Matching algorithm](hard-coded-defi/fair-batch-matching.md) page or the [FairBatchMatching.pdf](https://github.com/CoinFuMasterShifu/FairBatchMatching/blob/main/FairBatchMatching.pdf) paper. Try it interactively at [warthog.network/defi-demo](https://warthog.network/defi-demo).

### Status

Fair Batch Matching is **deployed on the testnet** (`core/defi` branch) alongside the rest of Native DeFi.

---

## Browser Nodes

Warthog is the first cryptocurrency project to ship a full node that runs entirely in the browser. The native C&#43;&#43; node is compiled to WebAssembly, and the entire chain database (~1.3 GB) is persisted in the browser's Origin Private File System (OPFS) as a SQLite file. Close the tab, reopen it tomorrow — your node is still synced, still validating, still part of the network.

- **No installation** — open a website, run a node.
- **No firewall configuration** — browser nodes connect outbound to bridge nodes over WebSocket.
- **No chain wipe on refresh** — OPFS persists across sessions.
- **~1.3 GB storage** — same as a normal full node, just in your browser.

Today browser nodes connect through bridge nodes. The next step is **P2P between browsers** over WebRTC, with a signaling protocol for peer discovery, NAT traversal, and handshake negotiation. When P2P ships, two tabs in the same room can sync the chain between each other — no server in the middle — turning every browser into a first-class network peer and removing the centralizing dependency on bridge nodes.

### Status

The browser-node engine is largely implemented and runs against bridge nodes today. The WebRTC P2P layer is on the roadmap but currently **on hold** — DeFi features have top priority. The protocol design for the future P2P layer is documented at [WebRTC Protocol](browser-nodes/webrtc-protocol.md). See the full [Browser Nodes Introduction](browser-nodes/Introduction.md) for architecture details.

---

## Where to go next

- New to Warthog: [Wart-dapp (GUI wallet)](../guides/wallet/wart-dapp.md) or [Mining quick start](../guides/mining/quickstart.md).
- DeFi: [DeFi demo](https://warthog.network/defi-demo) or [Fair Batch Matching algorithm](hard-coded-defi/fair-batch-matching.md).
- Run a node: [Running a Warthog node](../guides/node/node.md).
- Looking for the math? The PoBW and FBM papers are linked from the [Whitepaper & Research](https://warthog.network#research) section of the website.
