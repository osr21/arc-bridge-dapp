# Arc Bridge dApp

> Browser-based USDC bridge between Ethereum testnets and **Arc Testnet** powered by Circle's Cross-Chain Transfer Protocol (CCTP).

[![Live Demo](https://img.shields.io/badge/demo-live-brightgreen)](https://arc-bridge.replit.app)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.9-blue)](https://www.typescriptlang.org/)

---

## Overview

Arc Bridge lets users move USDC natively across chains — no wrapped tokens, no custodians.
Your USDC is **burned** on the source chain and **minted fresh** on the destination via Circle's attestation service.

| Feature | Description |
|---------|-------------|
| 🌉 **USDC Bridge** | Sepolia / Base Sepolia / Arb Sepolia / OP Sepolia ↔ Arc Testnet |
| ⛽ **Gas Dashboard** | Live Arc gas price, USDC balance, faucet links |
| 🖼 **NFT Bridge** | Lock-and-mint UI for ERC-721 tokens (requires contract deployment) |
| 📜 **Transaction History** | Persistent log with burn + mint explorer links |

---

## Quick Start

```bash
# 1. Clone
git clone https://github.com/osr21/arc-bridge-dapp.git
cd arc-bridge-dapp

# 2. Install dependencies (requires pnpm)
pnpm install

# 3. Set environment secrets
#    VITE_ALCHEMY_API_KEY — Alchemy API key for reliable RPC
#    SESSION_SECRET       — Express session secret (API server)

# 4. Run the bridge frontend
pnpm --filter @workspace/bridge-dapp run dev

# 5. (Optional) Run the API server
pnpm --filter @workspace/api-server run dev
```

Open the URL printed by Vite. Connect MetaMask (or any injected wallet), select chains, enter an amount, and bridge.

---

## Architecture

```
artifacts/
├── bridge-dapp/          # React + Vite frontend (main dApp)
│   └── src/
│       ├── pages/        # home · gas · nft · history · settings
│       ├── lib/          # store · constants · cctp · fee · wallet-proxy
│       └── components/   # shadcn/ui, onboarding modal
├── api-server/           # Express 5 backend (shared infra, unused by bridge)
└── mockup-sandbox/       # Component preview server (Canvas / design work)
```

See [docs/architecture.md](docs/architecture.md) for a detailed breakdown.

---

## Bridge Flow

```
User clicks Bridge
       │
       ▼
1. Fee Transfer ──► transfer(treasury, fee) on source chain  ← MetaMask prompt 1
       │
       ▼
2. Approve ──────► USDC.approve(cctpBurnContract, amount)    ← MetaMask prompt 2
       │
       ▼
3. Burn ─────────► MessageTransmitter burns USDC             ← MetaMask prompt 3
       │
       ▼
4. Attest ───────► Circle Iris API polls for attestation (~10–20 s)
       │
       ▼
5. Mint ─────────► MessageTransmitter mints USDC on destination
```

---

## Supported Networks

| Network | Chain ID | USDC Contract |
|---------|----------|---------------|
| Ethereum Sepolia | 11155111 | `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238` |
| Base Sepolia | 84532 | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` |
| Arbitrum Sepolia | 421614 | `0x75faf114eafb1BDbe2F0316DF893fd58CE46AA4d` |
| Optimism Sepolia | 11155420 | `0x5fd84259d66Cd46123540766Be93DFE6D43130D7` |
| **Arc Testnet** | **5042002** | `0x3600000000000000000000000000000000000000` |

---

## Protocol Fee

A 0.30% fee (minimum $0.25 USDC) is collected on every bridge as a plain ERC-20 transfer to the treasury wallet **before** the CCTP flow begins. This ensures the fee settles on-chain regardless of bridge outcome.

- **Rate:** 30 bps (0.30%), floor $0.25 USDC
- **NFT Bridge:** $0.50 flat per bridge
- **Treasury:** `0xdb5019b8DfbccEF8906C39B16a4870082eAbBc4C`

---

## Stack

- **Frontend:** React 18, Vite, TypeScript 5.9
- **Wallet:** wagmi v3, RainbowKit v2, viem v2
- **Bridge:** @circle-fin/app-kit, @circle-fin/adapter-viem-v2
- **State:** Zustand (persisted to localStorage)
- **UI:** shadcn/ui, Tailwind CSS, lucide-react
- **RPC:** Alchemy (via `VITE_ALCHEMY_API_KEY`) with read-proxy transport
- **Backend:** Express 5, PostgreSQL, Drizzle ORM

---

## License

MIT © 2025 osr21
