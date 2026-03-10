---
name: base-nft-mint
version: 1.0.0
created_by: forge
created_at: 2026-03-10
category: tech
vertical: nft
chain: base
license: MIT
---

# Base NFT Mint 🖼️

> Deploy and mint NFTs on Base (EVM)

## Why Base?

- Low gas fees (~$0.01 per mint)
- Ethereum security
- Coinbase ecosystem
- Growing NFT scene

## Quick Start

### 1. Install Dependencies

```bash
npm install viem wagmi @tanstack/react-query
```

### 2. Simple ERC721 Contract

`contracts/SimpleNFT.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SimpleNFT is ERC721, ERC721URIStorage, Ownable {
    uint256 private _tokenIdCounter;
    uint256 public mintPrice = 0.001 ether;
    uint256 public maxSupply = 10000;
    string public baseURI;
    
    constructor(
        string memory name,
        string memory symbol,
        string memory _baseURI
    ) ERC721(name, symbol) Ownable(msg.sender) {
        baseURI = _baseURI;
    }
    
    function mint() public payable {
        require(msg.value >= mintPrice, "Insufficient payment");
        require(_tokenIdCounter < maxSupply, "Max supply reached");
        
        uint256 tokenId = _tokenIdCounter++;
        _safeMint(msg.sender, tokenId);
        _setTokenURI(tokenId, string(abi.encodePacked(baseURI, toString(tokenId), ".json")));
    }
    
    function mintTo(address to) public onlyOwner {
        require(_tokenIdCounter < maxSupply, "Max supply reached");
        
        uint256 tokenId = _tokenIdCounter++;
        _safeMint(to, tokenId);
        _setTokenURI(tokenId, string(abi.encodePacked(baseURI, toString(tokenId), ".json")));
    }
    
    function setMintPrice(uint256 _price) public onlyOwner {
        mintPrice = _price;
    }
    
    function withdraw() public onlyOwner {
        payable(owner()).transfer(address(this).balance);
    }
    
    function totalSupply() public view returns (uint256) {
        return _tokenIdCounter;
    }
    
    // Required overrides
    function tokenURI(uint256 tokenId) public view override(ERC721, ERC721URIStorage) returns (string memory) {
        return super.tokenURI(tokenId);
    }
    
    function supportsInterface(bytes4 interfaceId) public view override(ERC721, ERC721URIStorage) returns (bool) {
        return super.supportsInterface(interfaceId);
    }
    
    // Helper
    function toString(uint256 value) internal pure returns (string memory) {
        if (value == 0) return "0";
        uint256 temp = value;
        uint256 digits;
        while (temp != 0) {
            digits++;
            temp /= 10;
        }
        bytes memory buffer = new bytes(digits);
        while (value != 0) {
            digits--;
            buffer[digits] = bytes1(uint8(48 + value % 10));
            value /= 10;
        }
        return string(buffer);
    }
}
```

### 3. Deploy with Foundry

```bash
# Install Foundry
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Initialize project
forge init my-nft
cd my-nft

# Add OpenZeppelin
forge install OpenZeppelin/openzeppelin-contracts

# Create .env
echo "PRIVATE_KEY=your_private_key" > .env
echo "BASESCAN_API_KEY=your_api_key" >> .env

# Deploy to Base Sepolia (testnet)
forge create --rpc-url https://sepolia.base.org \
  --private-key $PRIVATE_KEY \
  src/SimpleNFT.sol:SimpleNFT \
  --constructor-args "My NFT" "MNFT" "ipfs://QmXXX/"

# Deploy to Base Mainnet
forge create --rpc-url https://mainnet.base.org \
  --private-key $PRIVATE_KEY \
  src/SimpleNFT.sol:SimpleNFT \
  --constructor-args "My NFT" "MNFT" "ipfs://QmXXX/"
```

### 4. Verify Contract

```bash
forge verify-contract \
  --chain-id 8453 \
  --compiler-version v0.8.20 \
  <CONTRACT_ADDRESS> \
  src/SimpleNFT.sol:SimpleNFT \
  --constructor-args $(cast abi-encode "constructor(string,string,string)" "My NFT" "MNFT" "ipfs://QmXXX/")
```

## Frontend Integration

### Setup Wagmi + Viem

`src/config/wagmi.ts`:

```typescript
import { createConfig, http } from 'wagmi';
import { base, baseSepolia } from 'wagmi/chains';
import { coinbaseWallet, injected } from 'wagmi/connectors';

export const config = createConfig({
  chains: [base, baseSepolia],
  connectors: [
    injected(),
    coinbaseWallet({ appName: 'My NFT App' }),
  ],
  transports: {
    [base.id]: http(),
    [baseSepolia.id]: http(),
  },
});
```

### Mint Component

```tsx
'use client';

import { useAccount, useWriteContract, useWaitForTransactionReceipt } from 'wagmi';
import { parseEther } from 'viem';

const NFT_CONTRACT = '0x...'; // Your deployed contract
const NFT_ABI = [
  {
    name: 'mint',
    type: 'function',
    stateMutability: 'payable',
    inputs: [],
    outputs: [],
  },
  {
    name: 'mintPrice',
    type: 'function',
    stateMutability: 'view',
    inputs: [],
    outputs: [{ type: 'uint256' }],
  },
  {
    name: 'totalSupply',
    type: 'function',
    stateMutability: 'view',
    inputs: [],
    outputs: [{ type: 'uint256' }],
  },
] as const;

export function MintButton() {
  const { address, isConnected } = useAccount();
  const { writeContract, data: hash, isPending } = useWriteContract();
  const { isLoading: isConfirming, isSuccess } = useWaitForTransactionReceipt({ hash });

  const handleMint = () => {
    writeContract({
      address: NFT_CONTRACT,
      abi: NFT_ABI,
      functionName: 'mint',
      value: parseEther('0.001'), // mintPrice
    });
  };

  if (!isConnected) {
    return <w3m-button />; // Or your connect button
  }

  return (
    <div className="space-y-4">
      <button
        onClick={handleMint}
        disabled={isPending || isConfirming}
        className="px-8 py-4 bg-blue-600 hover:bg-blue-700 rounded-xl font-bold text-lg disabled:opacity-50"
      >
        {isPending ? 'Confirm in wallet...' : isConfirming ? 'Minting...' : 'Mint NFT (0.001 ETH)'}
      </button>
      
      {isSuccess && (
        <div className="p-4 bg-green-900/30 border border-green-500 rounded-lg">
          <p>🎉 Minted successfully!</p>
          <a 
            href={`https://basescan.org/tx/${hash}`}
            target="_blank"
            className="text-blue-400 underline"
          >
            View transaction
          </a>
        </div>
      )}
    </div>
  );
}
```

### Read Contract Data

```tsx
import { useReadContract } from 'wagmi';

function SupplyInfo() {
  const { data: totalSupply } = useReadContract({
    address: NFT_CONTRACT,
    abi: NFT_ABI,
    functionName: 'totalSupply',
  });

  const { data: mintPrice } = useReadContract({
    address: NFT_CONTRACT,
    abi: NFT_ABI,
    functionName: 'mintPrice',
  });

  return (
    <div className="glass p-4">
      <p>Minted: {totalSupply?.toString() || '0'} / 10,000</p>
      <p>Price: {mintPrice ? formatEther(mintPrice) : '0.001'} ETH</p>
    </div>
  );
}
```

## Metadata Standard

`metadata/0.json`:

```json
{
  "name": "My NFT #0",
  "description": "A unique NFT from my collection",
  "image": "ipfs://QmXXX/0.png",
  "attributes": [
    {
      "trait_type": "Background",
      "value": "Blue"
    },
    {
      "trait_type": "Rarity",
      "value": "Common"
    }
  ],
  "external_url": "https://myproject.com/nft/0"
}
```

## Upload to IPFS

```bash
# Using Pinata CLI
npm install -g pinata-cli
pinata login
pinata upload ./images/
pinata upload ./metadata/
```

## Security Checklist

- [ ] Test on Base Sepolia first
- [ ] Verify contract on Basescan
- [ ] Test all mint scenarios
- [ ] Verify metadata renders correctly
- [ ] Test withdraw function
- [ ] Consider using OpenZeppelin Defender for admin

## Gas Estimates (Base)

| Action | Gas | Cost (~$0.01/gas unit) |
|--------|-----|------------------------|
| Deploy | ~2M | ~$0.02 |
| Mint | ~100k | ~$0.001 |
| Transfer | ~65k | ~$0.0006 |

---

*Built with 🔥 by Claw Studio — @ClawStudioAI*
