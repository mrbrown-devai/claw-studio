---
name: ton-connect-kit
version: 1.0.0
created_by: forge
created_at: 2026-03-10
category: tech
vertical: general
chain: ton
license: MIT
---

# TON Connect Kit 🔗

> Simple wallet connection for TON apps

## Installation

```bash
npm install @tonconnect/ui-react
```

## Setup

### 1. Create Manifest

`public/tonconnect-manifest.json`:

```json
{
  "url": "https://your-app.com",
  "name": "Your App Name",
  "iconUrl": "https://your-app.com/icon-192.png",
  "termsOfUseUrl": "https://your-app.com/terms",
  "privacyPolicyUrl": "https://your-app.com/privacy"
}
```

### 2. Create Provider

`src/providers/TonConnectProvider.tsx`:

```tsx
'use client';

import { TonConnectUIProvider } from '@tonconnect/ui-react';

const manifestUrl = process.env.NEXT_PUBLIC_APP_URL + '/tonconnect-manifest.json';

export function TonConnectProvider({ children }: { children: React.ReactNode }) {
  return (
    <TonConnectUIProvider 
      manifestUrl={manifestUrl}
      actionsConfiguration={{
        twaReturnUrl: 'https://t.me/YourBotUsername/app'
      }}
    >
      {children}
    </TonConnectUIProvider>
  );
}
```

### 3. Wrap App

`src/app/layout.tsx`:

```tsx
import { TonConnectProvider } from '@/providers/TonConnectProvider';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <TonConnectProvider>
          {children}
        </TonConnectProvider>
      </body>
    </html>
  );
}
```

## Usage

### Connect Button (Built-in)

```tsx
import { TonConnectButton } from '@tonconnect/ui-react';

function Header() {
  return (
    <header className="flex justify-between items-center p-4">
      <h1>My App</h1>
      <TonConnectButton />
    </header>
  );
}
```

### Custom Connect Button

```tsx
'use client';

import { useTonConnectUI, useTonWallet } from '@tonconnect/ui-react';

function CustomConnectButton() {
  const [tonConnectUI] = useTonConnectUI();
  const wallet = useTonWallet();

  if (wallet) {
    return (
      <div className="flex items-center gap-3">
        <span className="font-mono text-sm">
          {wallet.account.address.slice(0, 6)}...{wallet.account.address.slice(-4)}
        </span>
        <button
          onClick={() => tonConnectUI.disconnect()}
          className="px-4 py-2 bg-red-600 rounded-lg text-sm"
        >
          Disconnect
        </button>
      </div>
    );
  }

  return (
    <button
      onClick={() => tonConnectUI.openModal()}
      className="px-6 py-3 bg-blue-600 hover:bg-blue-700 rounded-lg font-medium"
    >
      Connect Wallet
    </button>
  );
}
```

### Check Connection Status

```tsx
import { useTonWallet, useTonConnectUI } from '@tonconnect/ui-react';

function WalletInfo() {
  const wallet = useTonWallet();
  const [tonConnectUI] = useTonConnectUI();

  if (!wallet) {
    return <p>Not connected</p>;
  }

  return (
    <div className="space-y-2">
      <p><strong>Address:</strong> {wallet.account.address}</p>
      <p><strong>Chain:</strong> {wallet.account.chain}</p>
      <p><strong>Wallet:</strong> {wallet.device.appName}</p>
    </div>
  );
}
```

### Send Transaction

```tsx
import { useTonConnectUI, useTonWallet } from '@tonconnect/ui-react';
import { toNano } from '@ton/ton';

function SendTransaction() {
  const [tonConnectUI] = useTonConnectUI();
  const wallet = useTonWallet();

  async function send() {
    if (!wallet) {
      await tonConnectUI.openModal();
      return;
    }

    const transaction = {
      validUntil: Math.floor(Date.now() / 1000) + 600,
      messages: [
        {
          address: 'UQD...recipient...',
          amount: toNano('0.1').toString(),
          // Optional: add comment
          payload: Buffer.from('Hello!').toString('base64'),
        },
      ],
    };

    try {
      const result = await tonConnectUI.sendTransaction(transaction);
      console.log('Success:', result.boc);
    } catch (error) {
      console.error('Failed:', error);
    }
  }

  return (
    <button onClick={send} className="px-6 py-3 bg-green-600 rounded-lg">
      Send 0.1 TON
    </button>
  );
}
```

### Listen to Connection Changes

```tsx
import { useTonConnectUI } from '@tonconnect/ui-react';
import { useEffect } from 'react';

function ConnectionListener() {
  const [tonConnectUI] = useTonConnectUI();

  useEffect(() => {
    const unsubscribe = tonConnectUI.onStatusChange((wallet) => {
      if (wallet) {
        console.log('Connected:', wallet.account.address);
        // Track user, create session, etc.
      } else {
        console.log('Disconnected');
      }
    });

    return () => unsubscribe();
  }, [tonConnectUI]);

  return null;
}
```

## Hooks Reference

| Hook | Returns | Purpose |
|------|---------|---------|
| `useTonWallet()` | `Wallet \| null` | Current wallet state |
| `useTonConnectUI()` | `[TonConnectUI]` | UI control methods |
| `useTonAddress()` | `string \| null` | Just the address |
| `useIsConnectionRestored()` | `boolean` | Connection restored from storage |

## Supported Wallets

TON Connect 2.0 supports 20+ wallets including:
- Tonkeeper
- OpenMask
- MyTonWallet
- TON Wallet (Telegram)
- And more...

## Common Issues

### "Manifest not found"
- Check the manifest URL is accessible
- Ensure CORS headers allow access
- Use HTTPS in production

### "Transaction rejected"
- Check `validUntil` is in the future
- Verify addresses are valid
- Ensure sufficient balance

### "Connection not restored"
- Wait for `useIsConnectionRestored()` to be true before checking wallet

## Full Example

```tsx
'use client';

import { 
  TonConnectButton, 
  useTonWallet, 
  useTonConnectUI,
  useIsConnectionRestored 
} from '@tonconnect/ui-react';
import { toNano } from '@ton/ton';

export default function WalletPage() {
  const wallet = useTonWallet();
  const [tonConnectUI] = useTonConnectUI();
  const connectionRestored = useIsConnectionRestored();

  if (!connectionRestored) {
    return <div>Loading...</div>;
  }

  return (
    <div className="min-h-screen p-4 space-y-6">
      <header className="flex justify-between items-center">
        <h1 className="text-2xl font-bold">My TON App</h1>
        <TonConnectButton />
      </header>

      {wallet ? (
        <div className="glass p-6 space-y-4">
          <h2 className="text-xl">Connected Wallet</h2>
          <p className="font-mono text-sm break-all">
            {wallet.account.address}
          </p>
          <button
            onClick={async () => {
              await tonConnectUI.sendTransaction({
                validUntil: Math.floor(Date.now() / 1000) + 600,
                messages: [{
                  address: 'UQD...',
                  amount: toNano('0.1').toString(),
                }],
              });
            }}
            className="w-full py-3 bg-blue-600 rounded-lg"
          >
            Send 0.1 TON
          </button>
        </div>
      ) : (
        <div className="glass p-6 text-center">
          <p className="text-gray-400 mb-4">
            Connect your wallet to continue
          </p>
          <button
            onClick={() => tonConnectUI.openModal()}
            className="px-6 py-3 bg-blue-600 rounded-lg"
          >
            Connect Wallet
          </button>
        </div>
      )}
    </div>
  );
}
```

---

*Built with 🔥 by Claw Studio — @ClawStudioAI*
