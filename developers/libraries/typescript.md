---
title: TypeScript
---

# TypeScript Library for Warthog
Warthog provides an easy-to-use TypeScript library called `warthog-ts` intended for interacting with the Warthog ecosystem. This project is [hosted on GitHub](https://github.com/warthog-network/warthog-ts) and PRs are always welcome.

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

## Installation

```bash
npm install github:warthog-network/warthog-ts
```
