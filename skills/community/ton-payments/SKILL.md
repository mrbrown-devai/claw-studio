---
name: ton-payments
version: 1.0.0
created_by: forge
created_at: 2026-03-10
category: tech
vertical: general
chain: ton
license: MIT
---

# TON Payments 💰

> Accept Stars, TON, and USDT in your Telegram Mini App

## Payment Options

| Method | Best For | Fees |
|--------|----------|------|
| ⭐ Telegram Stars | Micro-payments, in-app purchases | ~30% |
| 💎 TON | Crypto-native users | Gas only |
| 💵 USDT (TON) | Stable payments | Gas only |

## 1. Telegram Stars ⭐

### Backend: Create Invoice

```typescript
// src/app/api/stars/invoice/route.ts
import { NextResponse } from 'next/server';

const BOT_TOKEN = process.env.TELEGRAM_BOT_TOKEN!;

export async function POST(req: Request) {
  const { userId, amount, title, description } = await req.json();

  const response = await fetch(
    `https://api.telegram.org/bot${BOT_TOKEN}/createInvoiceLink`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        title,
        description,
        payload: JSON.stringify({ userId, amount }),
        currency: 'XTR', // Telegram Stars
        prices: [{ label: title, amount }], // amount in Stars
      }),
    }
  );

  const data = await response.json();
  
  if (!data.ok) {
    return NextResponse.json({ error: data.description }, { status: 400 });
  }

  return NextResponse.json({ invoiceLink: data.result });
}
```

### Frontend: Open Payment

```tsx
'use client';

import { invoice } from '@telegram-apps/sdk-react';

async function buyWithStars(amount: number) {
  // Get invoice link from backend
  const res = await fetch('/api/stars/invoice', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      userId: window.Telegram?.WebApp?.initDataUnsafe?.user?.id,
      amount,
      title: 'Premium Access',
      description: 'Unlock all features',
    }),
  });
  
  const { invoiceLink } = await res.json();
  
  // Open Telegram payment UI
  const result = await invoice.open(invoiceLink, 'url');
  
  if (result === 'paid') {
    // Payment successful!
    console.log('Payment completed');
  }
}
```

### Webhook: Handle Success

```typescript
// src/app/api/telegram/webhook/route.ts
import { NextResponse } from 'next/server';

export async function POST(req: Request) {
  const update = await req.json();

  if (update.pre_checkout_query) {
    // Always approve (or validate first)
    await fetch(
      `https://api.telegram.org/bot${process.env.TELEGRAM_BOT_TOKEN}/answerPreCheckoutQuery`,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          pre_checkout_query_id: update.pre_checkout_query.id,
          ok: true,
        }),
      }
    );
  }

  if (update.message?.successful_payment) {
    const payment = update.message.successful_payment;
    const payload = JSON.parse(payment.invoice_payload);
    
    // Grant access to user
    await grantPremiumAccess(payload.userId);
  }

  return NextResponse.json({ ok: true });
}
```

## 2. TON Payments 💎

### Send TON Transaction

```tsx
'use client';

import { useTonConnectUI, useTonWallet } from '@tonconnect/ui-react';
import { toNano } from '@ton/ton';

function PayWithTON() {
  const [tonConnectUI] = useTonConnectUI();
  const wallet = useTonWallet();

  async function pay(amountTON: number, recipientAddress: string) {
    if (!wallet) {
      await tonConnectUI.openModal();
      return;
    }

    const transaction = {
      validUntil: Math.floor(Date.now() / 1000) + 600, // 10 min
      messages: [
        {
          address: recipientAddress,
          amount: toNano(amountTON).toString(),
        },
      ],
    };

    try {
      const result = await tonConnectUI.sendTransaction(transaction);
      console.log('TX Hash:', result.boc);
      
      // Verify on backend
      await verifyPayment(result.boc);
    } catch (e) {
      console.error('Payment failed:', e);
    }
  }

  return (
    <button 
      onClick={() => pay(1, 'UQD...')}
      className="glass px-6 py-3 font-medium"
    >
      Pay 1 TON 💎
    </button>
  );
}
```

### Backend: Verify Transaction

```typescript
// src/app/api/ton/verify/route.ts
import { TonClient, Address } from '@ton/ton';
import { NextResponse } from 'next/server';

const client = new TonClient({
  endpoint: 'https://toncenter.com/api/v2/jsonRPC',
});

export async function POST(req: Request) {
  const { boc, expectedAmount, recipientAddress } = await req.json();

  // Parse and verify the transaction
  // In production: check the actual on-chain state
  
  // For now, trust the BOC (implement proper verification)
  return NextResponse.json({ verified: true });
}
```

## 3. USDT Payments 💵

### Send USDT (Jetton) Transaction

```tsx
'use client';

import { useTonConnectUI, useTonWallet } from '@tonconnect/ui-react';
import { Address, beginCell, toNano } from '@ton/ton';

// USDT Jetton Master on TON Mainnet
const USDT_MASTER = 'EQCxE6mUtQJKFnGfaROTKOt1lZbDiiX1kCixRv7Nw2Id_sDs';

function PayWithUSDT() {
  const [tonConnectUI] = useTonConnectUI();
  const wallet = useTonWallet();

  async function payUSDT(amountUSDT: number, recipientAddress: string) {
    if (!wallet) {
      await tonConnectUI.openModal();
      return;
    }

    // Get user's USDT wallet address (jetton wallet)
    const userJettonWallet = await getJettonWalletAddress(
      wallet.account.address,
      USDT_MASTER
    );

    // Build transfer message
    const forwardPayload = beginCell()
      .storeUint(0, 32) // op = 0 (simple transfer)
      .storeStringTail('Payment for service')
      .endCell();

    const body = beginCell()
      .storeUint(0xf8a7ea5, 32) // op: jetton transfer
      .storeUint(0, 64) // query_id
      .storeCoins(amountUSDT * 1e6) // USDT has 6 decimals
      .storeAddress(Address.parse(recipientAddress))
      .storeAddress(Address.parse(wallet.account.address)) // response_destination
      .storeBit(0) // no custom payload
      .storeCoins(toNano('0.05')) // forward_ton_amount
      .storeBit(1) // forward_payload in ref
      .storeRef(forwardPayload)
      .endCell();

    const transaction = {
      validUntil: Math.floor(Date.now() / 1000) + 600,
      messages: [
        {
          address: userJettonWallet,
          amount: toNano('0.1').toString(), // Gas
          payload: body.toBoc().toString('base64'),
        },
      ],
    };

    try {
      const result = await tonConnectUI.sendTransaction(transaction);
      console.log('USDT TX:', result.boc);
    } catch (e) {
      console.error('USDT payment failed:', e);
    }
  }

  return (
    <button 
      onClick={() => payUSDT(10, 'UQD...')}
      className="glass px-6 py-3 font-medium"
    >
      Pay 10 USDT 💵
    </button>
  );
}

// Helper: Get jetton wallet address
async function getJettonWalletAddress(
  ownerAddress: string,
  jettonMaster: string
): Promise<string> {
  // Call get_wallet_address on jetton master
  // Implementation depends on your setup
  return '...';
}
```

## Complete Payment Component

```tsx
'use client';

import { useState } from 'react';

type PaymentMethod = 'stars' | 'ton' | 'usdt';

export function PaymentSelector({ 
  amountUSD,
  onSuccess 
}: { 
  amountUSD: number;
  onSuccess: () => void;
}) {
  const [method, setMethod] = useState<PaymentMethod>('stars');
  const [loading, setLoading] = useState(false);

  // Convert USD to each currency
  const prices = {
    stars: Math.ceil(amountUSD * 50), // ~$0.02 per star
    ton: amountUSD / 5, // Example: $5/TON
    usdt: amountUSD,
  };

  return (
    <div className="glass p-6 space-y-4">
      <h3 className="text-lg font-semibold">Choose Payment Method</h3>
      
      <div className="grid grid-cols-3 gap-2">
        {(['stars', 'ton', 'usdt'] as const).map((m) => (
          <button
            key={m}
            onClick={() => setMethod(m)}
            className={`p-3 rounded-lg border transition ${
              method === m 
                ? 'border-blue-500 bg-blue-500/20' 
                : 'border-white/10 hover:border-white/30'
            }`}
          >
            {m === 'stars' && '⭐'}
            {m === 'ton' && '💎'}
            {m === 'usdt' && '💵'}
            <div className="text-xs mt-1 text-gray-400">
              {m === 'stars' && `${prices.stars} Stars`}
              {m === 'ton' && `${prices.ton.toFixed(2)} TON`}
              {m === 'usdt' && `${prices.usdt} USDT`}
            </div>
          </button>
        ))}
      </div>

      <button
        onClick={() => handlePayment(method, prices[method])}
        disabled={loading}
        className="w-full py-3 bg-blue-600 hover:bg-blue-700 rounded-lg font-medium disabled:opacity-50"
      >
        {loading ? 'Processing...' : `Pay ${amountUSD} USD`}
      </button>
    </div>
  );
}
```

## Best Practices

1. **Always verify payments on backend** — Never trust client-side
2. **Use webhooks for Stars** — Required for production
3. **Handle failed transactions** — Show clear error messages
4. **Store transaction history** — For support and debugging
5. **Test on testnet first** — Use TON testnet before mainnet

## Need More?

- Custom payment flows → Contact Claw Studio
- Escrow systems → Contact Claw Studio
- Subscription payments → Contact Claw Studio

---

*Built with 🔥 by Claw Studio — @ClawStudioAI*
