---
name: solana-token-deploy
version: 1.0.0
created_by: forge
created_at: 2026-03-10
category: tech
vertical: defi
chain: solana
license: MIT
---

# Solana Token Deploy 🪙

> Deploy an SPL Token on Solana in minutes

## Prerequisites

- Node.js 18+
- Solana CLI installed
- Some SOL for fees (~0.05 SOL)

## Quick Start (CLI)

### 1. Install Solana Tools

```bash
# Install Solana CLI
sh -c "$(curl -sSfL https://release.solana.com/v1.18.4/install)"

# Install SPL Token CLI
cargo install spl-token-cli
```

### 2. Create Wallet (if needed)

```bash
# Generate new keypair
solana-keygen new --outfile ~/.config/solana/devnet.json

# Set as default
solana config set --keypair ~/.config/solana/devnet.json

# Set to devnet (use mainnet-beta for production)
solana config set --url devnet

# Airdrop some SOL (devnet only)
solana airdrop 2
```

### 3. Create Token

```bash
# Create the token mint
spl-token create-token

# Output: Creating token <MINT_ADDRESS>

# Create token account
spl-token create-account <MINT_ADDRESS>

# Mint initial supply (1 billion with 9 decimals)
spl-token mint <MINT_ADDRESS> 1000000000
```

### 4. Add Metadata (Metaplex)

```bash
# Install Metaplex CLI
npm install -g @metaplex-foundation/cli

# Create metadata
metaplex tokens create-metadata \
  --mint <MINT_ADDRESS> \
  --name "My Token" \
  --symbol "MTK" \
  --uri "https://example.com/metadata.json"
```

## Programmatic Deployment

### Install Dependencies

```bash
npm install @solana/web3.js @solana/spl-token @metaplex-foundation/mpl-token-metadata
```

### Create Token Script

`scripts/create-token.ts`:

```typescript
import {
  Connection,
  Keypair,
  PublicKey,
  Transaction,
  sendAndConfirmTransaction,
} from '@solana/web3.js';
import {
  createMint,
  getOrCreateAssociatedTokenAccount,
  mintTo,
  TOKEN_PROGRAM_ID,
} from '@solana/spl-token';
import {
  createCreateMetadataAccountV3Instruction,
  PROGRAM_ID as METADATA_PROGRAM_ID,
} from '@metaplex-foundation/mpl-token-metadata';
import * as fs from 'fs';

async function createToken() {
  // Connect to cluster
  const connection = new Connection('https://api.devnet.solana.com', 'confirmed');
  
  // Load wallet
  const secretKey = JSON.parse(
    fs.readFileSync(process.env.WALLET_PATH || '~/.config/solana/devnet.json', 'utf-8')
  );
  const payer = Keypair.fromSecretKey(new Uint8Array(secretKey));
  
  console.log('Wallet:', payer.publicKey.toString());
  console.log('Balance:', await connection.getBalance(payer.publicKey) / 1e9, 'SOL');

  // Token config
  const decimals = 9;
  const initialSupply = 1_000_000_000; // 1 billion

  // 1. Create the mint
  console.log('\n1. Creating token mint...');
  const mint = await createMint(
    connection,
    payer,           // Payer
    payer.publicKey, // Mint authority
    payer.publicKey, // Freeze authority (null to disable)
    decimals
  );
  console.log('Mint:', mint.toString());

  // 2. Create token account for payer
  console.log('\n2. Creating token account...');
  const tokenAccount = await getOrCreateAssociatedTokenAccount(
    connection,
    payer,
    mint,
    payer.publicKey
  );
  console.log('Token Account:', tokenAccount.address.toString());

  // 3. Mint initial supply
  console.log('\n3. Minting initial supply...');
  await mintTo(
    connection,
    payer,
    mint,
    tokenAccount.address,
    payer,
    initialSupply * (10 ** decimals)
  );
  console.log('Minted:', initialSupply.toLocaleString(), 'tokens');

  // 4. Add metadata
  console.log('\n4. Adding metadata...');
  const [metadataPDA] = PublicKey.findProgramAddressSync(
    [
      Buffer.from('metadata'),
      METADATA_PROGRAM_ID.toBuffer(),
      mint.toBuffer(),
    ],
    METADATA_PROGRAM_ID
  );

  const metadata = {
    name: 'My Token',
    symbol: 'MTK',
    uri: 'https://example.com/metadata.json', // Upload to IPFS/Arweave
    sellerFeeBasisPoints: 0,
    creators: null,
    collection: null,
    uses: null,
  };

  const createMetadataIx = createCreateMetadataAccountV3Instruction(
    {
      metadata: metadataPDA,
      mint: mint,
      mintAuthority: payer.publicKey,
      payer: payer.publicKey,
      updateAuthority: payer.publicKey,
    },
    {
      createMetadataAccountArgsV3: {
        data: metadata,
        isMutable: true,
        collectionDetails: null,
      },
    }
  );

  const tx = new Transaction().add(createMetadataIx);
  await sendAndConfirmTransaction(connection, tx, [payer]);
  console.log('Metadata added!');

  // Summary
  console.log('\n=== TOKEN CREATED ===');
  console.log('Mint:', mint.toString());
  console.log('Decimals:', decimals);
  console.log('Supply:', initialSupply.toLocaleString());
  console.log('Metadata:', metadataPDA.toString());
  console.log('\nView on Solscan:');
  console.log(`https://solscan.io/token/${mint.toString()}?cluster=devnet`);
}

createToken().catch(console.error);
```

### Metadata JSON

Upload to IPFS/Arweave:

```json
{
  "name": "My Token",
  "symbol": "MTK",
  "description": "A community token for...",
  "image": "https://arweave.net/xxx",
  "external_url": "https://mytoken.com",
  "attributes": [],
  "properties": {
    "category": "currency"
  }
}
```

## Token Operations

### Transfer Tokens

```typescript
import { transfer } from '@solana/spl-token';

await transfer(
  connection,
  payer,
  sourceTokenAccount,
  destinationTokenAccount,
  payer,
  amount * (10 ** decimals)
);
```

### Burn Tokens

```typescript
import { burn } from '@solana/spl-token';

await burn(
  connection,
  payer,
  tokenAccount,
  mint,
  payer,
  amount * (10 ** decimals)
);
```

### Revoke Mint Authority (Lock Supply)

```typescript
import { setAuthority, AuthorityType } from '@solana/spl-token';

await setAuthority(
  connection,
  payer,
  mint,
  payer,
  AuthorityType.MintTokens,
  null // Remove authority = locked supply
);
```

## Security Checklist

- [ ] Test on devnet first
- [ ] Verify metadata URI is permanent (IPFS/Arweave)
- [ ] Consider revoking mint authority for trust
- [ ] Consider revoking freeze authority
- [ ] Verify token appears correctly on explorers
- [ ] Keep keypair backup secure

## Common Issues

### "Insufficient funds"
- Get SOL: `solana airdrop 2` (devnet) or buy SOL (mainnet)

### "Token not showing metadata"
- Metadata takes time to index
- Verify URI is accessible
- Check metadata format matches standard

### "Authority errors"
- Ensure you're using the correct keypair
- Check authority hasn't been revoked

## Mainnet Deployment

1. Switch to mainnet: `solana config set --url mainnet-beta`
2. Fund wallet with real SOL
3. Run the same commands
4. Verify on Solscan

---

*Built with 🔥 by Claw Studio — @ClawStudioAI*
