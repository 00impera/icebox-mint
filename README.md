# ❄ IceBox NFT

> **On-chain mystery box protocol on Monad Mainnet.**
> 11 tiers · 8 materials · random vault prizes · 100,000 GEMS jackpot

[![Monad Mainnet](https://img.shields.io/badge/network-Monad%20Mainnet-7c6fff?style=flat-square)](https://monad.xyz)
[![Solidity](https://img.shields.io/badge/solidity-0.8.20-00ffe7?style=flat-square)](https://soliditylang.org)
[![License: MIT](https://img.shields.io/badge/license-MIT-ff00c8?style=flat-square)](LICENSE)
[![Verified](https://img.shields.io/badge/sourcify-verified-39ff14?style=flat-square)](https://sourcify-api-monad.blockvision.org)

---

## Contract

| | |
|---|---|
| **Address** | `0x307416a34f911e85Cf2cBF0C0aF3527491B40002` |
| **Chain** | Monad Mainnet (Chain ID: 143) |
| **Verification** | ✅ Exact match — [Sourcify](https://sourcify-api-monad.blockvision.org/v2/verify/a63ab032-0f19-4494-9089-6625df2533a2) |
| **Explorer** | [explorer.monad.xyz](https://explorer.monad.xyz/address/0x307416a34f911e85Cf2cBF0C0aF3527491B40002) |
| **Standard** | ERC-721 |
| **Compiler** | Solc 0.8.20 + `via_ir = true` |

---

## Overview

IceBox is a fully on-chain mystery box NFT. Users mint a box at one of 11 price tiers, then open it on-chain to reveal a random prize. Every open draws from a shared token vault and awards GEMS tokens based on the prize tier.

All metadata and SVG art is generated entirely on-chain — no IPFS, no centralized API.

---

## Tiers & Materials

| Tier | Material | Price | Jackpot Odds |
|------|----------|-------|--------------|
| 0 | Rock | 100 MON | 3% |
| 1 | Silver | 200 MON | 3% |
| 2 | Gold | 300 MON | 4% |
| 3 | Fire | 400 MON | 4% |
| 4 | Crystal | 500 MON | 5% |
| 5 | Platinum | 1,000 MON | 6% |
| 6 | Premier | 2,000 MON | 7% |
| 7 | Ultra | 3,000 MON | 8% |
| 8 | Ultra | 4,000 MON | 9% |
| 9 | Ultra | 5,000 MON | 10% |
| 10 | Ultra | 10,000 MON | 12% |

Higher tiers get better odds across all prize categories — less empty, more jackpot.

---

## Prize Tiers

| Prize | GEMS Reward | Rarity |
|-------|------------|--------|
| Empty | 0 | — |
| Small | 100 GEMS | Common |
| Medium | 1,000 GEMS | Uncommon |
| Big | 10,000 GEMS | Rare |
| Jackpot | 100,000 GEMS + random vault token | Legendary |

Every non-empty prize also triggers a random ERC-20 token draw from the vault. The amount scales with tier via `TOKEN_BPS`.

---

## Vault Tokens

The contract holds a prize vault of ERC-20 tokens that are distributed on wins:

| Token | Address |
|-------|---------|
| mUSDC | `0xA9873347059379E4db1111A716f0E00Cde26C5dF` |
| mUSD | `0xf77C5C1a904813Fd63871B2Bb6A36268EDee2c8A` |
| PENDLE | `0xf2CFFF86c7E8DF74c7335Af23CeA9428a740201B` |
| GEMS | `0x49931887171BF46922b2b80Aa834537A80C50B70` |
| WETH | `0x39AAF7DF83D7C1Fb6E22a5666A4bA57E62A19246` |

---

## Repository Structure

```
mock-usdc/
├── src/
│   └── IceBoxNFT_v2.sol      # Main contract
├── script/
│   └── DeployIceBoxV2.s.sol  # Foundry deploy script
├── frontend/
│   └── icebox-mint.html      # Single-file mint UI
├── foundry.toml               # Foundry config (via_ir enabled)
└── README.md
```

---

## Deploy

### Prerequisites

- [Foundry](https://book.getfoundry.sh/getting-started/installation)
- Monad RPC endpoint
- Funded wallet with MON

### Setup

```bash
git clone https://github.com/YOUR_USERNAME/icebox-nft
cd icebox-nft
forge install
```

### Configure

Add to `foundry.toml`:

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc = "0.8.20"
via_ir = true
optimizer = true
optimizer_runs = 200

[rpc_endpoints]
monad = "https://rpc.monad.xyz"
```

### Build & Deploy

```bash
export PRIVATE_KEY=0xYOUR_PRIVATE_KEY

forge build

forge script script/DeployIceBoxV2.s.sol \
  --rpc-url monad \
  --private-key $PRIVATE_KEY \
  --broadcast
```

### Verify

```bash
forge verify-contract \
  --rpc-url https://rpc.monad.xyz \
  --verifier sourcify \
  --verifier-url 'https://sourcify-api-monad.blockvision.org/' \
  --chain-id 143 \
  --compiler-version 0.8.20 \
  --via-ir \
  YOUR_CONTRACT_ADDRESS \
  src/IceBoxNFT_v2.sol:IceBoxNFT
```

---

## Post-Deploy Setup

After deploying, run these steps to activate the contract:

```bash
# 1. Enable minting
cast send $CONTRACT "setMintOpen(bool)" true \
  --rpc-url monad --private-key $PRIVATE_KEY

# 2. Add vault tokens
cast send $CONTRACT "addVaultTokensBatch(address[])" \
  "[0xA987...,0xf77C...,0xf2CF...,0x4993...,0x39AA...]" \
  --rpc-url monad --private-key $PRIVATE_KEY

# 3. Link GEMS token so jackpot payouts work
cast send $GEMS_TOKEN "setIceBoxContract(address)" $CONTRACT \
  --rpc-url monad --private-key $PRIVATE_KEY

# 4. Deposit prize tokens into vault
cast send $CONTRACT "depositToken(address,uint256)" \
  $TOKEN_ADDRESS $AMOUNT \
  --rpc-url monad --private-key $PRIVATE_KEY
```

---

## Key Functions

### User

```solidity
// Mint a box — send exact MON value for the tier
function mint(uint8 tier) external payable returns (uint256 tokenId)

// Open a box you own to reveal the prize
function openBox(uint256 tokenId) external

// Read box data
function getBox(uint256 tokenId) external view returns (BoxData memory)

// On-chain SVG metadata
function tokenURI(uint256 tokenId) external view returns (string memory)
```

### Owner

```solidity
function setMintOpen(bool open) external onlyOwner
function ownerMint(address to, uint8 tier, uint256 qty) external onlyOwner
function addVaultToken(address token) external onlyOwner
function depositToken(address token, uint256 amount) external onlyOwner
function withdrawMON() external onlyOwner
function setBoxPrice(uint8 tier, uint256 price) external onlyOwner
function setGemsReward(uint8 prizeTier, uint256 amount) external onlyOwner
```

---

## How Randomness Works

Randomness is derived on-chain using `keccak256` of:

```solidity
// At mint — seed generation
keccak256(abi.encodePacked(block.timestamp, block.prevrandao, tokenId, to, tier))

// At open — prize roll
keccak256(abi.encodePacked(box.seed, block.timestamp, block.prevrandao, tokenId))
```

> ⚠️ This is pseudo-random and sufficient for a game with low financial stakes. For high-value applications consider integrating a VRF oracle.

---

## On-Chain SVG Art

Each box state renders a unique SVG stored fully on-chain:

- **Unopened** — material-colored box with animated sparkles
- **Empty** — dark greyed-out box with X mark
- **Small / Medium / Big** — gem shape in rarity color
- **Jackpot** — gold burst with rays and coin symbols

No external dependencies. Metadata is returned as a `data:application/json;base64` URI directly from `tokenURI()`.

---

## Frontend

A zero-dependency single HTML file mint UI is included at `frontend/icebox-mint.html`. It connects to MetaMask, auto-switches to Monad Mainnet, and handles the full mint flow.

To run locally:

```bash
open frontend/icebox-mint.html
# or
python3 -m http.server 8080
```

To deploy: upload to any static host — GitHub Pages, Vercel, Cloudflare Pages.

---

## License

MIT — see [LICENSE](LICENSE)

---

## Built on

- [Monad](https://monad.xyz) — high-performance EVM blockchain
- [Foundry](https://book.getfoundry.sh) — smart contract development toolkit
- [OpenZeppelin](https://openzeppelin.com/contracts) — ERC-721, Ownable, ReentrancyGuard, Base64
