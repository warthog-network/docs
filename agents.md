# AGENTS.md — Warthog Network docs

Documentation site for the Warthog Network cryptocurrency, built with [Retype](https://retype.com/) and published at https://docs.warthog.network. **Part of a multi-repo project hub** — see `../HUB.md` (at the hub root) for the full sub-repo layout, public APIs, and cross-repo source-of-truth mapping.

This repo is **content-only** — there is no application code, no `package.json`, no test suite, and no local linter. An agent that looks for `npm test`, `pytest`, `go build`, etc. will find nothing.

## Network context (mainnet vs testnet)

Warthog runs two networks with different feature sets. When documenting anything DeFi-related, be explicit which network is being described:

| Network | Branch | Features |
|---------|--------|----------|
| **Mainnet** | `core/master` | `wartTransfer` only (no DeFi) |
| **Testnet** | `core/defi` | All 7 DeFi transaction types, Fair Batch Matching, pools, limit swaps |

The API surface, transaction types, and matcher logic are different on each branch. Do not import DeFi concepts (FBM, pools, `tokenTransfer`, etc.) into pages about mainnet behavior.

## Key constants (must match HUB.md and source repos)

If a page here contradicts any of these, fix the page, not the constant.

- **Chain ID:** `0x539` (decimal `1337`) — used in browser-wallet integration examples.
- **Block version:** `4`
- **Verus version:** `v2.2`
- **Currency:** `1 WART = 100,000,000 E8` (8 decimals)
- **Block time:** `20s`
- **Initial reward:** `3 WART`
- **HD derivation path:** `m/44'/2070'/0'/0/{index}` (NOT `m/44'/2070'/0/0/{index}` — the third segment is hardened)
- **Stratum port (mining):** `3456` (by convention)

## Source-of-truth per page

When editing a specific page, cross-check against the original source rather than other docs pages:

- **Transaction schema (7 signed DeFi types)** — `core/defi/src/shared/src/communication/create_transaction.hpp` and `developers/api/rest/transactions.md`. Common prefix: `pinHash(32) + pinHeight(4) + nonceId(4) + reserved(3) + compactFee(2)` = 45 bytes. Signed = user-created; Implicit = node-generated (reward, match).
- **Address format** — 48 hex chars + 4-byte checksum (52 hex total).
- **FBM (Fair Batch Matching)** — `unique-features/hard-coded-defi/fair-batch-matching.md` is itself the canonical doc; do not fork it elsewhere. FBM is the only fair matching for hybrid pool+order-book liquidity (Nash equilibrium: same conversion price, no remaining buy/sell pair).
- **Janushash / PoBW** — `unique-features/janushash/`; live paper at https://warthog.network/PoBW.pdf.
- **Public Data API** — `guides/public-data/index.md` must match what `data.warthog.network` actually serves. Canonical endpoint list lives in HUB.md `### Public Data API` (7 endpoints: legacy-nodes, defi-nodes, addresses, sha256t-hashrates, verushash2_2-hashrates, assets, assets/{hash}/info.json).
- **Browser-wallet integration** — `guides/browser-wallet-integration.md` references `window.warthog` with EIP-1193-style methods (`requestAccounts`, `transferWart`, `chainId`) and error codes (`4001`, `4100`, `-32603`, `-32601`); keep aligned with `warthog-network/browser-wallet`.

## Brand identity

The site uses the Warthog brand. When producing new assets or touching the theme:

- **Primary yellow:** `#FDB913` (Pantone `1375 C`). Logo SVG tokens are a strict subset: `st0` `#F8F8F9`, `st1` `#FDB913`, `st2` `#231F20`, `st3` `#FFFFFF`.
- **Logos:** pick from `../brand-kit/SVGs/<Variant>/` (`Circle`, `Short`, `Stacked`, `Ticker`) and a color scheme (`Black`, `Yellow`, `White`, `BW`, `Negative`, `Negative Yellow`).
- **Font:** Montserrat only (Bold for titles, Family for text blocks). The brand-kit provides `Fonts/Montserrat.zip`.
- **Full reference:** `../brand-kit/Guideline/WT_Guidelines.pdf` (V01.A, 2023·2024, by BalkyBot). See HUB.md `## Brand Kit` for the full palette, SVG tokens, and naming conventions.

`retype.yml` controls the site branding (`title`, `label`, `logo`, header links, footer). If the logo, title, or footer needs updating, edit it there — not in `.retype/` (build output, gitignored).

## Known inconsistencies to watch for

Several factual claims here can drift from the source. Check HUB.md `## Known Inconsistencies` before editing:

- **Node URL lists** — five divergent locations across sub-repos (`warthog-ts/KNOWN_NODES`, `mobile-wallet`, `node-gui`, `public-data/legacy-nodes.csv`, `public-data/defi-nodes.csv`). `data.warthog.network` JSON is canonical for current data.
- **Version drift** — `warthog-ts` has been ahead of `core/defi` in past releases; the published SDK version may not match the deployed node.
- **Docker tag collision** — all three `core/*` worktrees push the same `zzzjulien/warthog_node` image.

## Publishing

- Default branch is **`master`** (not `main`). Every push to `master` triggers `.github/workflows/retype-action.yml`, which builds the site and force-pushes the rendered output to a separate `retype` branch on GitHub via `retypeapp/action-github-pages` with `update-branch: true`.
- No local build is required. If you want to preview changes locally, install Retype (`npm install retypeapp` then `npx retype start`) — Retype is a paid product, so check with the maintainer before adding a license.
- The build output goes to `.retype/` (gitignored). Do not commit it.
- The published site is configured in `retype.yml` (`url`, `branding`, `edit.repo`, header links). If you change the GitHub repo URL, branding, or nav links, edit that file.

## Authoring rules

Retype parses every `.md` file at the repo root and below (except those in underscore-prefixed folders — see below). Conventions seen across the existing pages:

- **Front matter** at the top of every published page. Common keys:
  - `title:` — page title shown in the sidebar.
  - `label:` — alternate short label.
  - `order:` — controls sidebar position. Lower numbers appear first; siblings without `order` are alphabetised after numbered ones.
  - Directory-level ordering can also be set in a sibling `index.yml` (see `guides/index.yml`, `developers/index.yml`, `unique-features/janushash/index.yml`).
- **Admonitions** use Retype's `!!!` block:
  ```
  !!!
  Warning text here.
  !!!
  ```
  See `guides/mining/quickstart.md` for a working example.
- **Emoji shortcodes** like `:warning:`, `:information_source:` render in admonitions. Don't paste raw Unicode warning symbols where a shortcode is already in use in the file.
- **Images** live under `img/` and are referenced by absolute path (`/img/...`), which Retype resolves at the site root. Subfolders used today: `img/dapp/`, `img/extension/`, `img/get-started/`. Add new images to a logical subfolder, not the root.
- **Links** to external projects (pools, exchanges, miners, wallets) go in `links.md`, not scattered through guide pages. That file is the single source of truth for community listings.
- **Code blocks** don't need language tags, but fenced ```` ```bash ````, ```` ```javascript ````, etc. are used throughout — match the surrounding file's style.
- **Color values in this repo** should match the brand palette (`#FDB913`, `#E9E9E9` backgrounds, etc.) — see HUB.md `## Brand Kit` for the full list.

## Folder layout

Published content:
- `guides/` — user-facing guides (mining, node setup, wallets, public-data contributions, browser-wallet integration).
- `developers/` — API reference (`developers/api/` for REST/WebSocket, `_agents/` for agent-style notes), integration guides (`integration/` for miners/pools/wallets), and language libraries (`libraries/`).
- `unique-features/` — protocol-level deep dives (Janushash, hard-coded DeFi, browser nodes). These are the original-research pages; they get updated in place. FBM lives here as the canonical reference.
- `img/` — committed image assets.
- `readme.md` — landing page shown at `/`.
- `links.md` — community/external links directory.
- `retype.yml` — site config.

Not published (Retype skips underscore-prefixed paths):
- `_drafts/` — work-in-progress guides that haven't been promoted yet. Don't link to these from published pages.
- `_roadmap/` — planning notes.
- `_whitepaper.md`, `_version_history.md` — historical/reference docs kept out of the nav.
- Files like `unique-features/janushash/Janushash.md.old` — prior revisions; don't revive content from these without checking with a maintainer.

## Repository boundaries (do not confuse)

This repo **only documents** other repos. It does not host any of the following:

- **Public data** (legacy-nodes, defi-nodes, addresses, hashrates, assets) — lives in `warthog-network/public-data` and is served from `data.warthog.network`. The `guides/public-data/` folder is documentation *about* that repo, not the data itself. To edit the actual data, edit that other repo (and possibly the corresponding JSON on `data.warthog.network`).
- **Node binaries** — `warthog-network/core` (testnet: `defi` branch; mainnet: `master` branch).
- **Wallets** — `warthog-network/browser-wallet`, `warthog-network/mobile-wallet`, `warthog-network/Wartlock`, `warthog-network/wart-wallet`, `warthog-network/wart-dapp`.
- **Miners** — `warthog-network/janusminer` (GPU/CPU), `warthog-network/bzminer` (third-party build).
- **Explorer** — `warthog-network/client-explorer`.
- **Website** — `warthog-network/website`.
- **Brand assets** — `warthog-network/brand-kit` (logo SVGs, fonts, guidelines).
- **Whitepaper** — `warthog-network/whitepaper`; live PDF at https://github.com/warthog-network/whitepaper/releases/download/build/main.pdf.

If a task is to change any of those, the work belongs in the corresponding sub-repo, not here.

## Style and content notes

- `readme.md` carries a banner stating the docs are partially AI-augmented and asks contributors to report inconsistencies at https://github.com/warthog-network/docs/issues. Before adding new AI-generated content, double-check factual claims against the source repos or whitepaper (`_whitepaper.md`, `unique-features/janushash/Janushash.md`, `unique-features/hard-coded-defi/fair-batch-matching.md`).
- Mining numbers and recommendations change: bzminer version floors, block reward, stratum port (`3456` by convention), chain ID (`0x539` = 1337, used in the browser-wallet integration examples). Verify before publishing.
- Public-data API endpoints documented in `guides/public-data/index.md` must match what `data.warthog.network` actually serves.
- The browser-wallet integration page (`guides/browser-wallet-integration.md`) is referenced as `window.warthog`; keep the `requestAccounts` / `transferWart` / `chainId` method names and EIP-1193-style error codes (`4001`, `4100`, `-32603`, `-32601`) consistent with the extension's actual implementation in `warthog-network/browser-wallet`.

## Verification checklist for non-trivial edits

There is no automated test. For non-trivial edits, verify by:

1. Reading the existing page end-to-end before changing structure — Retype is sensitive to front matter formatting (a missing `---` will silently break the page).
2. Cross-checking claims against the source repo or whitepaper (see "Source-of-truth per page" above for which source applies to which page).
3. Checking that all internal links resolve to files that exist (Retype renders broken links as broken nav).
4. Confirming image paths start with `/img/` and the file actually exists at that path.
5. Confirming any new constants (chain IDs, ports, byte sizes) match the "Key constants" section above.
