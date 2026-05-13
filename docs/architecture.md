# Architecture

## Overview

Arc Bridge is a **pure-frontend dApp** — all CCTP orchestration happens client-side via Circle's App Kit SDK. The Express API server is shared infrastructure included for future use (auth, analytics) but is not required for bridging.

## Monorepo Structure

```
pnpm-workspace.yaml          workspace root
├── artifacts/
│   ├── bridge-dapp/         @workspace/bridge-dapp  (React + Vite)
│   ├── api-server/          @workspace/api-server   (Express 5)
│   └── mockup-sandbox/      @workspace/mockup-sandbox
├── lib/                     shared TypeScript libraries
├── scripts/                 utility scripts
└── tsconfig.base.json       strict shared TS config
```

## Key Modules

### `src/lib/constants.ts`
Single source of truth for:
- Chain configurations (RPC, explorer, CCTP domain, bridgeChain object)
- USDC contract addresses per chain
- Protocol fee constants and `calcProtocolFee()` helper
- Treasury address

### `src/lib/store.ts`
Zustand store with `persist` middleware (localStorage key `arc-bridge-history`).
- `BridgeTransaction` — id, txHash, mintTxHash, fromChain, toChain, amount, status, type
- `updateTransactionStatus(id, status, hash?, mintTxHash?)` — used after CCTP mint confirms

### `src/lib/wallet-proxy.ts`
**Read-proxy transport** — the definitive fix for MetaMask's built-in Blast API RPC on Sepolia.

MetaMask injects its own Sepolia RPC and ignores `wallet_addEthereumChain` for built-in networks. This proxy intercepts all read methods (`eth_getTransactionCount`, `eth_call`, `eth_estimateGas`, etc.) and routes them through an Alchemy-backed public client, while write/sign calls (`eth_sendTransaction`, `personal_sign`) still go through the MetaMask transport.

```
MetaMask transport
    │
    └─ write methods (sendTransaction, sign) ──► MetaMask ──► user prompt
    └─ read methods  (getBalance, getCode …)  ──► Alchemy public client
```

### `src/lib/cctp.ts`
CCTP attestation utilities:
- `extractMessageHashFromReceipt()` — parses `MessageSent` event logs
- `pollIrisAttestation(messageHash)` — polls Circle's Iris API until attestation is ready

### `src/lib/fee.ts`
On-chain fee collection before CCTP bridging:
- `sendFeeTransfer(walletClient, fromChainId, feeUsdc)` — ERC-20 `transfer(treasury, fee)`
- `feeExplorerUrl(fromChainId, txHash)` — constructs chain-specific explorer URL

### `src/lib/hooks/use-cctp-polling.ts`
React hook that polls pending transactions from the Zustand store every 15 seconds and updates their status when the mint confirms.

## Data Flow

```
HomePage.handleBridge()
  │
  ├─ calcProtocolFee(amount)              [constants.ts]
  ├─ sendFeeTransfer(wallet, chain, fee)  [fee.ts]       ← MetaMask #1
  ├─ kit.bridge({ from, to, amount })     [Circle SDK]
  │    ├─ Approve                                         ← MetaMask #2
  │    ├─ Burn                                            ← MetaMask #3
  │    ├─ FetchAttestation               [cctp.ts / Iris API]
  │    └─ Mint
  ├─ addTransaction(...)                  [store.ts]
  └─ setBridgeSuccessData(...)            [React state]
```

## RPC Strategy

| Call type | Route | Reason |
|-----------|-------|--------|
| Read (eth_call, getBalance, …) | Alchemy | Reliable, no Blast API noise |
| Write (sendTransaction) | MetaMask | User signs in wallet |
| Arc Testnet reads | Direct RPC (https://rpc.testnet.arc.network) | Not on Alchemy |

## State Management

All bridge state is ephemeral React state except for transaction history, which is persisted to localStorage. No server-side session is needed — the dApp is fully self-contained.

```
localStorage['arc-bridge-history']
  → { state: { transactions: BridgeTransaction[] }, version: 0 }
```
