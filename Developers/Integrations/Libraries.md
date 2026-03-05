# Libraries

## `warthog-ts`
We are working on an official TypeScript library named [warthog-ts](https://github.com/warthog-network/warthog-ts) that greatly simplifies interaction with the Warthog ecosystem.

### Features

- Generate wallets and sign transactions
- Built-in API client for node communication
- Works in Node.js, React Native, and Browsers

### Quick Start

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

### Installation

```bash
npm install github:warthog-network/warthog-ts
```
