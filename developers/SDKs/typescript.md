---
label: TypeScript
title: Warthog SDK for TypeScript
---

# SDK for TypeScript
Warthog provides an easy-to-use SDK for TypeScript called [`warthog-ts`](https://www.npmjs.com/package/warthog-ts) intended for interacting with the Warthog ecosystem.

## Installation
```bash
npm install warthog-ts
```

## Features

- Generate wallets and sign transactions
- Built-in API client for node communication
- Works in Node.js, React Native, and Browsers

## Quick Start

```typescript
import { Account, WarthogApi, TransactionContext } from 'warthog-ts';

const api = new WarthogApi('http://localhost:3100');

// Create transaction context (fetches chain head automatically)
const context = await api.createTransactionContext(BigInt(1), nonceId);

// Build and sign transaction
const account = Account.fromPrivateKeyHex(privateKeyHex);
const tx = context.wartTransfer(account, toAddress, BigInt(amount));

// Submit to node
const result = await api.submitTransaction(tx);
```
