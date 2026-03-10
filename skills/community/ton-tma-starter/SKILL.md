---
name: ton-tma-starter
version: 1.0.0
created_by: forge
created_at: 2026-03-10
category: tech
vertical: general
chain: ton
license: MIT
---

# TON TMA Starter 🚀

> Complete scaffold for Telegram Mini Apps on TON

## What You Get

A production-ready TMA template with:
- ✅ Next.js 15 + React 19
- ✅ TypeScript strict mode
- ✅ Tailwind CSS + glass morphism
- ✅ TON Connect wallet integration
- ✅ Telegram WebApp SDK
- ✅ Dark theme by default
- ✅ Mobile-first responsive

## Quick Start

### 1. Create Project

```bash
npx create-next-app@latest my-tma --typescript --tailwind --app
cd my-tma
```

### 2. Install Dependencies

```bash
npm install @tonconnect/ui-react @telegram-apps/sdk-react
npm install -D @telegram-apps/types
```

### 3. Setup TON Connect

Create `src/providers/TonProvider.tsx`:

```tsx
'use client';

import { TonConnectUIProvider } from '@tonconnect/ui-react';

const manifestUrl = 'https://your-domain.com/tonconnect-manifest.json';

export function TonProvider({ children }: { children: React.ReactNode }) {
  return (
    <TonConnectUIProvider manifestUrl={manifestUrl}>
      {children}
    </TonConnectUIProvider>
  );
}
```

### 4. Create Manifest

Create `public/tonconnect-manifest.json`:

```json
{
  "url": "https://your-domain.com",
  "name": "My TMA",
  "iconUrl": "https://your-domain.com/icon.png",
  "termsOfUseUrl": "https://your-domain.com/terms",
  "privacyPolicyUrl": "https://your-domain.com/privacy"
}
```

### 5. Setup Telegram SDK

Create `src/providers/TelegramProvider.tsx`:

```tsx
'use client';

import { useEffect, useState } from 'react';
import { init, miniApp, themeParams } from '@telegram-apps/sdk-react';

export function TelegramProvider({ children }: { children: React.ReactNode }) {
  const [ready, setReady] = useState(false);

  useEffect(() => {
    const initTelegram = async () => {
      try {
        init();
        miniApp.ready();
        
        // Apply Telegram theme
        if (themeParams.backgroundColor) {
          document.body.style.backgroundColor = themeParams.backgroundColor();
        }
        
        setReady(true);
      } catch (e) {
        // Not in Telegram context (dev mode)
        setReady(true);
      }
    };

    initTelegram();
  }, []);

  if (!ready) return null;
  return <>{children}</>;
}
```

### 6. Update Layout

Update `src/app/layout.tsx`:

```tsx
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import { TonProvider } from '@/providers/TonProvider';
import { TelegramProvider } from '@/providers/TelegramProvider';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'My TMA',
  description: 'Telegram Mini App on TON',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" className="dark">
      <body className={inter.className}>
        <TelegramProvider>
          <TonProvider>
            {children}
          </TonProvider>
        </TelegramProvider>
      </body>
    </html>
  );
}
```

### 7. Global Styles

Update `src/app/globals.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --background: #0a0a0a;
  --foreground: #fafafa;
}

body {
  background: var(--background);
  color: var(--foreground);
  min-height: 100vh;
  overflow-x: hidden;
}

/* Glass morphism utilities */
.glass {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
}

.glass-strong {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 16px;
}
```

### 8. Home Page with Wallet

Update `src/app/page.tsx`:

```tsx
'use client';

import { TonConnectButton, useTonWallet } from '@tonconnect/ui-react';

export default function Home() {
  const wallet = useTonWallet();

  return (
    <main className="min-h-screen p-4 flex flex-col items-center justify-center gap-6">
      <div className="glass p-8 text-center max-w-md w-full">
        <h1 className="text-2xl font-bold mb-4">My TMA</h1>
        <p className="text-gray-400 mb-6">
          Welcome to your Telegram Mini App
        </p>
        
        <TonConnectButton />
        
        {wallet && (
          <div className="mt-6 p-4 glass-strong text-left">
            <p className="text-sm text-gray-400">Connected:</p>
            <p className="font-mono text-sm truncate">
              {wallet.account.address}
            </p>
          </div>
        )}
      </div>
    </main>
  );
}
```

## Deploy to Vercel

```bash
npm run build
vercel deploy --prod
```

Then create your bot with @BotFather and set the Mini App URL.

## Next Steps

- Add payments → See `ton-payments` skill
- Add NFT minting → Contact Claw Studio
- Build full product → [claw.studio](https://claw.studio)

---

*Built with 🔥 by Claw Studio — @ClawStudioAI*
