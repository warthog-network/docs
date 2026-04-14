# API Documentation Agent Guide

!!!
This documentation was partially generated with AI augmentation. Report inconsistencies at [github.com/warthog-network/docs/issues](https://github.com/warthog-network/docs/issues).
!!!

## 1. Purpose

This directory contains the source-of-truth reference files for the Warthog node REST API. AI agents use these files to automatically generate, correct, and keep the API documentation up to date whenever the node is updated.

> **This file is a living document.** As new friction points are discovered during the update process, add them here to streamline future executions.

### Scope

All API-related documentation files must be kept in sync with `api.txt` and `schemas.json`. **Nothing shall be left stale.** Any example, field name, type, or structure that diverges from the schema is a bug. This includes but is not limited to:

- `rest.md` — overview table, endpoint examples, field names, types, anchors
- `rest/block-actions.md` — block body field structures
- `rest/create-transaction.md` — transaction field structures
- `rest/mempool-transactions.md` — mempool field structures (currently empty; pending live API verification)
- `websocket.md` — if applicable

When in doubt: schema wins. If `api.txt` or `schemas.json` changes, all affected API docs must be updated accordingly.

## 2. Reference Files

| File | Description | Source |
|------|-------------|--------|
| `api.txt` | Categorized list of all endpoints with C++ return types | Stripped version of `/debug/html_schemas` (links removed) |
| `schemas.json` | JSON array of all JSON schemas for return types | Raw output of `/debug/json_schemas` |
| `Agents.md` | This file — guide for AI agents | Manual |

## 3. Workflow Options

### Option A — Paste files (current)

1. User pastes `api.txt` and `schemas.json` into `_agents/`
2. AI agent reads these files to update documentation

### Option B — Direct API calls (future)

1. AI agent asks user for the RPC port (default `3000`)
2. AI agent calls `GET http://localhost:<port>/debug/json_schemas` → `schemas.json`
3. AI agent calls `GET http://localhost:<port>/debug/html_schemas` → `api.txt` (strip links)
4. AI agent uses response to update documentation

Both options produce identical content.

## 4. How to Interpret `api.txt`

`api.txt` is organized into category sections, each starting with a heading line:

```
API for Warthog node version v0.10.3 "a26330e"
Transaction Endpoints

    POST /transaction/add -> TransactionAddResult
    GET /transaction/mempool -> std::vector<MempoolEntry>
    ...
```

- **Version line** — The first line contains the node version and git hash. Update the version in `rest.md` to match this line.
- **Category headings** — Define the section order in the Overview table (e.g., Transaction Endpoints, Asset Endpoints, DEX Endpoints, Chart Endpoints). This order is authoritative.
- **Endpoint lines** — Format: `METHOD /path -> ReturnType`
  - `METHOD` is `GET`, `POST`, etc.
  - `:param` denotes a path parameter (e.g., `:txid`, `:account`, `:market`)
  - `?param=...` denotes a query parameter
  - `-> ReturnType` is the C++ return type (used to look up the schema in `schemas.json`)

## 5. How to Interpret `schemas.json`

`schemas.json` is a single-line JSON array (~42KB). To look up a schema, find the entry whose key matches the return type from `api.txt`.

### Schema Extraction Commands

Since `schemas.json` is a single line, use these Python commands to extract schemas:

**Look up a top-level schema by name:**
```bash
python3 -c "import json; d=json.load(open('schemas.json')); print(json.dumps(d.get('SchemaName'), indent=2))"
```

**Look up a `$defs` entry:**
```bash
python3 -c "import json; d=json.load(open('schemas.json')); print(json.dumps(d['\$defs'].get('TypeName'), indent=2))"
```

For example, `-> TransactionMinfeeResult` maps to:

```json
{
  "TransactionMinfeeResult": {
    "type": "object",
    "properties": {
      "minFee": { "$ref": "#/$defs/CompactFee" }
    },
    "additionalProperties": false
  }
}
```

### C++ types vs JSON types

The schemas use C++ type names as `$ref` targets. Common mappings:

| C++ type | JSON type |
|----------|-----------|
| `uint64_t` | non-negative integer (up to 2^53) |
| `uint32_t` | non-negative integer |
| `double` | number |
| `std::string` | string |
| `bool` | boolean |
| `Timestamp` | Unix timestamp (seconds) |
| `Height` | block height (uint32_t) |
| `FundsDecimal` | decimal string (e.g. `"100.500000"`) |
| `Wart` | `{ "E8": number, "str": string }` |
| `std::vector<X>` | array of `X` |
| `std::optional<X>` | `X` or `null` |
| `std::tuple<A,B,C>` | array `[A, B, C]` (fixed length) |
| `std::string` (return type only) | no schema — document as plain string |

To find a `$ref` target (e.g. `#/$defs/Wart`), search for the type name in `$defs`.

### Walking the `$defs` Chain

Many schemas use nested `$ref` targets. The agent must walk the chain to find the actual fields.

**Example: `TransactionMinfeeResult.minFee` → `CompactFee`**
1. Look up `TransactionMinfeeResult` → has `minFee: { "$ref": "#/$defs/CompactFee" }`
2. Look up `CompactFee` in `$defs` → has `{ "E8", "str", "bytes" }`
3. Result: `minFee` is an object with three fields

### Schema Key → Return Type Lookup

Not all schema keys match the C++ return type names from `api.txt` exactly:

| api.txt return type | schema key to look up |
|----------------------|-----------------------|
| `-> TransactionMinfeeResult` | `TransactionMinfeeResult` |
| `-> std::vector<MempoolEntry>` | `MempoolEntry` |
| `-> std::vector<BanEntry>` | `BanEntry` |
| `-> api::glaze::HashrateInfo` | `HashrateInfo` |
| `-> api::glaze::AccountHistory` | `api::glaze::AccountHistory` |
| `-> api::glaze::PeerinfoConnection` | `api::glaze::PeerinfoConnection` |
| `-> api::glaze::HeaderDownload` | `api::glaze::HeaderDownload` |
| `-> std::string` | no schema — document as "string" |
| `-> void` | no schema — no `data` returned |
| `-> double` | no schema — document as "number" |

For `std::vector<X>`, look up the item type `X`. For namespaces like `api::glaze::`, include the full path.

## 6. Response Wrapper Pattern

Every API endpoint returns an outer wrapper object:

**Success:**
```json
{
  "code": 0,
  "data": <inner return type from api.txt>
}
```

**Error:**
```json
{
  "code": <error code>,
  "error": <error message>
}
```

Documentation examples should show the inner type. The wrapper should not be repeated for every endpoint — it is implied.

## 7. Special Endpoints

Two endpoints in `api.txt` do not follow the standard wrapper pattern:

| Endpoint | Description |
|----------|-------------|
| `GET /debug/json_schemas` | Returns the raw content of `schemas.json` (no wrapper) |
| `GET /debug/html_schemas` | Returns HTML version of the endpoint list (no wrapper, includes links) |

These are source-of-truth endpoints used to generate the reference files. Do not document their return values in `rest.md` — they exist only for documentation tooling.

## 8. Variant Handling

Some return types use C++ `std::variant` for polymorphic results:

- `TransactionDetails.transaction` — variant of all transaction types (Reward, WartTransfer, TokenTransfer, NewOrder, Match, etc.)
- `MempoolEntry.transaction` — variant of all signed transaction types
- `SignedTransactionProcessed<A, P>` — wraps a processed transaction with data + processed result

**Rule:** Pick ONE concrete type for the documentation example (e.g., "Reward" for a reward transaction). Describe the variant briefly in the prose, but do not try to show all possible variants in the JSON. Use the most representative example.

## 9. Block-Actions Special Format

The file `rest/block-actions.md` documents block body transaction types using the **actual nested structure** from the schema. Do not flatten fields for readability — the schema structure is the canonical representation.

Every action entry follows this pattern:

```json
{
  "historyId": 123,
  "transaction": {
    "data": { ... },
    "hash": "...",
    "signedCommon": { ... },
    "processed": { ... }
  }
}
```

**Two categories of transactions:**
- **Unsigned** (result of block production, no signature): Reward, Match — have `data` and `hash`, no `signedCommon`
- **Signed** (submitted by users, require signature): Wart Transfer, Token Transfer, New Order, Liquidity Deposit, Liquidity Withdrawal, Asset Creation, Cancelation — have `data`, `hash`, `signedCommon`, and optionally `processed`

**When updating `block-actions.md`:**
- Always document the actual nested `{ historyId, transaction: { data, hash, signedCommon?, processed? } }` structure
- **REMOVE** action types that exist in the documentation but are NOT in the `BlockActions` schema (e.g. `orderCancelations` does not exist in the schema)
- **REMOVE** fields that exist in the documentation but NOT in the schema for a given action type
- For each action type, show the full nested JSON example with abbreviated values

## 10. Schema as Source of Truth

The schema is the source of truth. The `BlockActions` schema lists exactly 9 action types:

`reward`, `wartTransfers`, `tokenTransfers`, `newOrders`, `matches`, `liquidityDeposits`, `liquidityWithdrawals`, `assetCreations`, `cancelations`

Any action type not in this list should not be documented. Any field within an action type that is not in its schema should not be documented. When the schema and the live API output differ, the schema definition in `schemas.json` is authoritative for documentation purposes. If neither matches, ask the user for clarification or C++ source code (`tx_to_json` functions).

## 11. Rules for Updating `rest.md`

`rest.md` is the main REST API reference document. Follow these rules strictly:

### Overview table

- **MUST** reflect `api.txt` exactly — every endpoint present in `api.txt` must appear in the table, with correct path, method, and description
- **MUST** match the category order from `api.txt`
- **MUST** add new endpoints that appear in `api.txt` but not in `rest.md`
- **MUST** remove endpoints that are in `rest.md` but not in `api.txt`
- **MUST** rename endpoints that changed path (e.g. `/market/:market` → `/dex/market/:market`)
- **MUST** update the version string at the top to match `api.txt`

### Detailed sections

- **ALWAYS** correct stale examples to match the schema — never leave incorrect field names, types, or structures in place
- **Do not** replace detailed prose descriptions (e.g. Fair Batch Matching explanation), but update any stale field references or endpoint links within them
- **Add** new fields that exist in `schemas.json` but are missing from the docs
- **Augment** with information from `schemas.json` (structure, types, optional vs required)
- **REMOVE** fields that exist in the documentation but NOT in `schemas.json` — the schema is authoritative. Do not keep stale fields for backward compatibility or readability reasons.
- If a detailed section exists in `rest.md` but the endpoint is not in `api.txt`, **remove it**

### Query parameters

Group query parameters in a table. Do not list them inline in the description text.

### JSON examples

Use abbreviated JSON with `...` for truncated values. Focus on structure, not full data. Specific rules:
- Use abbreviated hex: `"hash": "4b3bc482..."` not the full 64-char value
- Use `...` or `[...]` for truncated arrays/objects
- Use realistic but generic values: `1`, `100`, `12345` for IDs
- Include ALL required fields; omit optional fields that are `null`

### Anchor links

Update all anchor links (`#anchor`) to match renamed endpoint paths. The format is:
- Strip leading `/`
- Replace `/` with `-`
- Replace `:` with empty string
- Example: `GET /dex/market/:market` → `#get-dexmarketmarket`

### Schema-derived example generation

When generating a new example from a schema:
1. Look up the return type schema from `api.txt`
2. Walk the `$defs` chain for all nested types
3. For each field: use the type to pick an appropriate abbreviated value
4. For arrays: show one representative entry, use `...` for the rest
5. For variants: pick one concrete type (see Section 8)
6. Compare against any existing example — if the structure differs significantly, the existing example is stale and should be replaced

## 12. Rules for Updating `block-actions.md`

`rest/block-actions.md` documents the fields of all block body transaction types. It must stay consistent with the `BlockActions` schema in `schemas.json`.

- Cross-check all block action types against their schemas in `schemas.json`
- Correct field names, types, and structures to match the schema
- Keep the existing human-readable descriptions and formatting — only correct structure
- If a schema defines a new field not yet in the docs, add it
- **REMOVE** action types documented in `block-actions.md` that are NOT present in the `BlockActions` schema — the schema is authoritative
- **REMOVE** fields within an action type that are documented but not in its schema
- Schema names (e.g. `SignedTransactionProcessed`) do not need to appear in the documentation — use them only to verify correctness
- Ask the user if the C++ source code (`tx_to_json` functions) is available, as it may contain additional context (naming, comments, invariants)

## 13. Documentation Conventions

When updating documentation, maintain these conventions:

- Directory names: lowercase with hyphens (e.g. `developers/api/rest/`, not `Developers/API/REST/`)
- Links to sibling or sub-directory files: relative to current file
- Links to files outside the current directory tree: relative to project root
- Retype link pattern: `[!ref Label](path.md)` for navigation
- Do not add comments in code examples
- Do not add emoji unless explicitly requested
- Fix stray formatting artifacts (e.g. stray code fences, broken lists) when editing sections

## 14. Step-by-Step Update Checklist

When `api.txt` or `schemas.json` changes, follow this sequence:

1. **Update version** — Top of `rest.md`: replace version string with the one from `api.txt` line 2

2. **Rebuild Overview table** — Compare `api.txt` line-by-line against the table:
   - Add rows for new endpoints
   - Remove rows for deleted endpoints
   - Rename paths that changed
   - Reorder rows to match `api.txt` category order

3. **Rename section anchors** — For each renamed endpoint, update its section heading anchor (`### `GET /old/path`` → `### `GET /new/path``)

4. **Remove stale sections** — If a detailed section exists for an endpoint NOT in `api.txt`, delete it entirely (e.g. `/market/:market/order/:historyId`)

5. **Update detailed sections** — For each endpoint in `api.txt`:
   a. Find its return type in `api.txt`
   b. Extract the schema from `schemas.json` using the extraction commands
   c. Walk `$defs` chain for nested types
   d. Compare schema against existing example in `rest.md`
   e. If example is stale (field names/structure don't match): rewrite it
   f. If example is missing: generate one from the schema
   g. For variants: pick one concrete type, describe others in prose

6. **Update `block-actions.md`** — Compare each action type against its schema:
   a. Verify field names and types match
   b. Verify example format (flattened, not nested)
   c. Add new fields if missing
   d. Keep existing descriptions — do not replace with schema-only text

7. **Verify no TODO placeholders remain** — Search for `TODO` in `rest.md` and `block-actions.md`; all should be replaced with schema-consistent examples

8. **Fix broken links** — After renaming endpoints, verify all cross-references are consistent:
   - Overview table links match section heading anchors
   - `block-actions.md` links are relative to `rest/`
   - `[!ref]` patterns use the correct Retype syntax

## 15. Asking Questions

If you encounter ambiguities during the update process, ask the user before proceeding. Examples of situations requiring clarification:

- A schema field has no clear human-readable name
- An endpoint has changed purpose but the new behavior is unclear
- A field in the schema appears to be internal (not meant for API consumers)
- The agent needs C++ source code to understand the exact semantics of a field
- There is conflicting information between `api.txt` and `schemas.json`
- The schema does not match any existing example and the agent is unsure which variant to document

## 16. Improving This Guide

This guide is a living document. After each update session, add friction points discovered during execution:

- Any schema that required special handling or interpretation
- Any endpoint whose schema was unclear or incomplete
- Any decision that required user input
- Any formatting or structural conventions that were not documented here

Add new friction points as numbered items in the appropriate section, or create new sections as needed.
