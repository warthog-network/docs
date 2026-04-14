# API Documentation Agent Guide

## 1. Purpose

This directory contains the source-of-truth reference files for the Warthog node REST API. AI agents use these files to automatically generate, correct, and keep the API documentation up to date whenever the node is updated.

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

`schemas.json` is a single-line JSON array. To look up a schema, find the entry whose key matches the return type from `api.txt`.

For example, `-> TransactionMinfeeResult` maps to:

```json
{
  "TransactionMinfeeResult": {
    "type": "object",
    "properties": {
      "minFeeE8": { "$ref": "#/$defs/Wart" }
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
| `Wart` | smallest unit of WART (uint64_t as string) |
| `std::vector<X>` | array of `X` |
| `std::optional<X>` | `X` or `null` |
| `std::tuple<A,B,C>` | array `[A, B, C]` (fixed length) |

To find a `$ref` target (e.g. `#/$defs/Wart`), search for `" Wart"` in `schemas.json`.

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

## 8. Rules for Updating `rest.md`

`rest.md` is the main REST API reference document. Follow these rules strictly:

### Overview table (lines 37–97)

- **MUST** reflect `api.txt` exactly — every endpoint present in `api.txt` must appear in the table, with correct path, method, and description
- **MUST** match the category order from `api.txt`
- **MUST** add new endpoints that appear in `api.txt` but not in `rest.md`
- **MUST** remove endpoints that are in `rest.md` but not in `api.txt` (e.g. `/market/:market/order/:historyId`)
- **MUST** rename endpoints that changed path (e.g. `/market/:market` → `/dex/market/:market`)
- **MUST** update the version string at the top to match `api.txt`

### Detailed sections

- **NEVER** replace detailed sections wholesale
- **Correct** incorrect field names, types, or descriptions using `schemas.json`
- **Add** new fields that exist in `schemas.json` but are missing from the docs
- **Augment** with information from `schemas.json` (structure, types, optional vs required)
- If a detailed section exists in `rest.md` but the endpoint is not in `api.txt`, **remove it**

### Query parameters

Group query parameters in a table. Do not list them inline in the description text.

### JSON examples

Use abbreviated JSON with `...` for truncated values. Focus on structure, not full data.

### Anchor links

Update all anchor links (`#anchor`) to match renamed endpoint paths.

## 9. Rules for Updating `block-transactions.md`

`block-transactions.md` documents the fields of all block body transaction types. It must stay consistent with the `BlockActions` schema in `schemas.json`.

- Cross-check all 10 block action types against their schemas
- Correct field names, types, and structures to match the schema
- Keep the existing human-readable descriptions and formatting — only correct structure
- If a schema defines a new field not yet in the docs, add it
- Schema names (e.g. `SignedTransactionProcessed`) do not need to appear in the documentation — use them only to verify correctness
- Ask the user if the C++ code that generated the schema is available, as it may contain additional context (naming, comments, invariants)

## 10. Documentation Conventions

When updating documentation, maintain these conventions:

- Directory names: lowercase with hyphens (e.g. `developers/api/rest/`, not `Developers/API/REST/`)
- Links to sibling or sub-directory files: relative to current file
- Links to files outside the current directory tree: relative to project root
- Retype link pattern: `[!ref Label](path.md)` for navigation
- Do not add comments in code examples
- Do not add emoji unless explicitly requested

## 11. Asking Questions

If you encounter ambiguities during the update process, ask the user before proceeding. Examples of situations requiring clarification:

- A schema field has no clear human-readable name
- An endpoint has changed purpose but the new behavior is unclear
- A field in the schema appears to be internal (not meant for API consumers)
- The agent needs C++ source code to understand the exact semantics of a field
- There is conflicting information between `api.txt` and `schemas.json`
