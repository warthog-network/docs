---
label: Asset Metadata
---

# Registering Asset Metadata

Warthog stores asset metadata in a database on the same VPS that
hosts the asset-metadata backend — not in a GitHub repository. This
keeps submission free of GitHub's content-moderation rules: the
backend accepts any image the asset's creator signs for, and the
project doesn't have to moderate third-party content to stay in good
standing on GitHub. The supported way to publish is the self-service
form at <https://testnet-assets.warthog.network/>.

## Prerequisites

- A Warthog **asset hash** (64 hex chars) for a token you created on
  the defi testnet (`core/defi` branch).
- The **private key of the asset's on-chain creator address**. The
  form proves authorship by signing a one-time challenge with this
  key and recovering the signer server-side; anyone else gets
  `403 not_owner`.
- (Optional) A 250×250 logo PNG and a 600×200 banner PNG.

## Submitting via the form

1. Open <https://testnet-assets.warthog.network/>.
2. Unlock a Warthog wallet in the page (saved wallet, wallet file,
   seed phrase, or private key). The private key never leaves your
   browser.
3. Type or paste the 64-hex asset hash. The page looks up the
   on-chain ticker and pre-fills it.
4. Fill in the long name, description, and any of the optional
   social links. Drop in a logo and (optionally) a banner. The form
   re-encodes images client-side to the exact dimensions the backend
   requires.
5. Click **Submit**. The form fetches a one-time challenge, signs
   it, POSTs the multipart form with your signature, and the
   backend stores the metadata in its database.

On success, the page shows the public URLs where your metadata will
be served. Updates to existing assets use the same form — re-submit
and the matching asset hash's row is overwritten.

## info.json schema

The backend stores each asset as a document matching this shape:

| Field         | Required | Notes |
|---------------|----------|-------|
| `hash`        | yes      | 64 hex chars — the asset hash |
| `ticker`      | yes      | 1–5 ASCII alphanumerics; taken from on-chain `AssetName`, **not** user-supplied |
| `name`        | yes      | ≤ 15 chars — human-readable long name |
| `description` | yes      | ≤ 500 chars |
| `website`     | no       | `https://` URL, ≤ 2048 chars |
| `telegram`    | no       | `https://` URL, ≤ 2048 chars |
| `discord`     | no       | `https://` URL, ≤ 2048 chars |
| `twitter`     | no       | `https://` URL, ≤ 2048 chars |

Plus optional `logo.png` (exactly 250×250 px) and `banner.png`
(exactly 600×200 px). All four URL fields must start with `https://`.

## API (programmatic submission)

For tooling that wants to submit metadata without going through the
form, the API reference is shipped with the frontend at
[`asset-metadata-frontend/docs/api.md`](https://github.com/warthog-network/asset-metadata-frontend/blob/master/docs/api.md).
The wire format is a 130-char hex signature (`r || s || recid`); see
[Wallet Integration](../../developers/integration/wallets) for the
signing scheme.

## Catalog endpoints

Once submitted, your metadata is served from the catalog host:

| Endpoint | Description |
|----------|-------------|
| <https://testnet-assets.warthog.network/assets.json> | Full catalog list |
| <https://testnet-assets.warthog.network/assets/\{hash\}/info.json> | Single asset metadata |
| <https://testnet-assets.warthog.network/assets/\{hash\}/logo.png> | Asset logo (250×250 PNG) |
| <https://testnet-assets.warthog.network/assets/\{hash\}/banner.png> | Asset banner (600×200 PNG) |

## Need help?

Join the Warthog community on
[Discord](https://discord.com/invite/QMDV8bGTdQ) for asset-creation
questions.