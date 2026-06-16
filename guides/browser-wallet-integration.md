# Browser Wallet Integration

This guide explains how to integrate your dApp with the Warthog Browser Wallet extension.

## Overview

The Warthog Browser Wallet is a Chrome extension that allows users to manage their WART tokens and interact with dApps. When installed, it exposes a `window.warthog` provider that websites can use to:

- Detect the wallet extension
- Request account access
- Send transactions (signed by the extension)
- Subscribe to account and chain changes

## Installation

Users install the Warthog Browser Wallet from the Chrome Web Store (link to be added) or by building from source.

## Detecting the Wallet

Before requesting account access, check if the wallet is installed:

```javascript
const isWalletInstalled = () => {
  return typeof window.warthog !== 'undefined';
};

if (isWalletInstalled()) {
  console.log('Warthog wallet is installed');
} else {
  console.log('Please install Warthog wallet');
}
```

## Provider API

The `window.warthog` provider implements an EIP-1193 inspired interface:

### Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `request({ method, params })` | Request object | Promise | Main entry point for all operations |
| `isConnected()` | None | boolean | Returns true if wallet is connected |
| `on(event, handler)` | event, handler function | unsubscribe function | Subscribe to events |
| `removeListener(event, handler)` | event, handler function | void | Unsubscribe from events |
| `disconnect()` | None | Promise<void> | Disconnect the wallet |

### Request Methods

#### `requestAccounts`

Requests the user to connect their wallet.

```javascript
try {
  const accounts = await window.warthog.request({ method: 'requestAccounts' });
  console.log('Connected accounts:', accounts);
} catch (error) {
  if (error.code === 4001) {
    console.log('User rejected the request');
  }
}
```

#### `accounts`

Returns the currently connected accounts without prompting the user.

```javascript
const accounts = await window.warthog.request({ method: 'accounts' });
```

#### `chainId`

Returns the current chain ID.

```javascript
const chainId = await window.warthog.request({ method: 'chainId' });
console.log('Chain ID:', chainId); // "0x539" (1337 in hex)
```

#### `transferWart`

Transfers WART tokens to a specified address.

```javascript
try {
  const txHash = await window.warthog.request({
    method: 'transferWart',
    params: {
      to: '0000000000000000000000000000000000000000000000000000000000000000de47c9b2',
      amount: '1.5' // WART amount as string
    }
  });
  console.log('Transaction hash:', txHash);
} catch (error) {
  console.error('Transfer failed:', error.message);
}
```

#### `sendTransaction`

Alias for `transferWart`. Same behavior.

```javascript
const txHash = await window.warthog.request({
  method: 'sendTransaction',
  params: {
    to: '0000000000000000000000000000000000000000000000000000000000000000de47c9b2',
    amount: '1.5'
  }
});
```

### Events

#### `accountsChanged`

Fired when the connected accounts change.

```javascript
const unsubscribe = window.warthog.on('accountsChanged', (accounts) => {
  console.log('Accounts changed:', accounts);
});

// Later, to unsubscribe:
unsubscribe();
```

#### `chainChanged`

Fired when the chain ID changes.

```javascript
const unsubscribe = window.warthog.on('chainChanged', (chainId) => {
  console.log('Chain changed to:', chainId);
});
```

### Error Codes

| Code | Name | Description |
|------|------|-------------|
| 4001 | User Rejected | User rejected the approval request |
| 4100 | Unauthorized | Not connected - call `requestAccounts` first |
| -32603 | Internal Error | Something went wrong in the wallet |
| -32601 | Method Not Found | The requested method doesn't exist |

## Example Integration

### React Hook

```javascript
import { useState, useEffect } from 'react';

function useWallet() {
  const [accounts, setAccounts] = useState([]);
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    if (!window.warthog) return;

    // Check initial connection
    window.warthog.request({ method: 'accounts' })
      .then(accounts => {
        setAccounts(accounts || []);
        setIsConnected(accounts && accounts.length > 0);
      });

    // Subscribe to changes
    const unsubAccounts = window.warthog.on('accountsChanged', (accounts) => {
      setAccounts(accounts || []);
      setIsConnected(accounts && accounts.length > 0);
    });

    return () => unsubAccounts();
  }, []);

  const connect = async () => {
    const accounts = await window.warthog.request({ method: 'requestAccounts' });
    setAccounts(accounts || []);
    setIsConnected(accounts && accounts.length > 0);
    return accounts;
  };

  return { accounts, isConnected, connect };
}
```

### React Component

```javascript
import { useWallet } from './hooks/useWallet';
import { transferWart } from './walletApi';

function SendButton() {
  const { accounts, connect } = useWallet();

  const handleSend = async () => {
    if (!accounts.length) {
      await connect();
    }

    try {
      const txHash = await transferWart(
        '0000000000000000000000000000000000000000000000000000000000000000de47c9b2',
        '1.5'
      );
      console.log('Success! TX:', txHash);
    } catch (error) {
      console.error('Failed:', error.message);
    }
  };

  return <button onClick={handleSend}>Send 1.5 WART</button>;
}
```

## React SDK (`wallet-sdk`)

The client-explorer includes a React SDK at `src/wallet-sdk/` that provides:

- `WalletProvider` - Context provider for wallet state
- `useWallet` - Hook for accessing wallet state
- `WalletButton` - Pre-built connect/send/disconnect button
- `SendModal` - Modal for sending WART transactions

### Usage

```javascript
import { WalletProvider, useWallet, WalletButton } from './wallet-sdk';

// Wrap your app
function App() {
  return (
    <WalletProvider>
      <YourApp />
    </WalletProvider>
  );
}

// Use in any component
function Header() {
  const { isConnected, accounts } = useWallet();

  return (
    <nav>
      <WalletButton />
      {isConnected && <span>{accounts[0]}</span>}
    </nav>
  );
}
```

## Security Notes

1. **Always check `isInstalled` before using `window.warthog`**
2. **Handle user rejection gracefully** - users can cancel at any time
3. **Validate addresses** before sending transactions
4. **Display transaction details** to users before they approve
5. **Never store private keys** - the wallet handles all signing

## Links

- [Warthog Browser Wallet Repository](https://github.com/warthog-network/browser-wallet)
- [Warthog Explorer](https://wartscan.io/)
- [Fair Batch Matching Paper](https://github.com/CoinFuMasterShifu/FairBatchMatching/blob/main/FairBatchMatching.pdf)
