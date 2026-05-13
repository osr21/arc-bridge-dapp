# Development Guide

## Prerequisites

| Tool | Version |
|------|---------|
| Node.js | 24.x |
| pnpm | 9.x |
| TypeScript | 5.9 |

## Setup

```bash
git clone https://github.com/osr21/arc-bridge-dapp.git
cd arc-bridge-dapp
pnpm install
```

## Environment Secrets

Create a `.env` file in `artifacts/bridge-dapp/` (never commit this):

```env
VITE_ALCHEMY_API_KEY=your_alchemy_key_here
```

Get a free Alchemy key at https://www.alchemy.com — any tier works for testnet.

The API server needs an additional secret:

```env
SESSION_SECRET=any_random_string_32_chars_or_more
```

## Running Locally

```bash
# Bridge frontend (Vite dev server)
pnpm --filter @workspace/bridge-dapp run dev

# API server (Express, port 5000)
pnpm --filter @workspace/api-server run dev
```

## Type Checking

```bash
# Full workspace typecheck
pnpm run typecheck

# Single package
pnpm --filter @workspace/bridge-dapp run typecheck
```

## Build

```bash
pnpm run build
```

> **Note:** `build` requires `PORT` and `BASE_PATH` env vars (injected by the Replit workflow system). For local verification, use `typecheck` instead.

## Project Scripts

| Command | Description |
|---------|-------------|
| `pnpm run typecheck` | Full workspace typecheck |
| `pnpm run typecheck:libs` | Typecheck shared libs only |
| `pnpm run build` | Build all packages |
| `pnpm --filter @workspace/api-spec run codegen` | Regenerate OpenAPI client after spec changes |

## Adding a New Chain

1. **`src/lib/constants.ts`** — add to `CHAIN_CONFIGS` and `USDC_ADDRESSES`
2. **`src/App.tsx`** — add to the wagmi chains array
3. **`src/pages/home.tsx`** — add chain ID to `evmChains`

## Wallet Proxy Transport

If you encounter RPC errors on Sepolia (Blast API 429s), the proxy transport in `src/lib/wallet-proxy.ts` handles this automatically by routing read calls through Alchemy. If Alchemy's key is not set, it falls back to the public Alchemy endpoint.

## Common Issues

| Issue | Solution |
|-------|----------|
| `VITE_ALCHEMY_API_KEY` not set | Vite falls back to the public Alchemy endpoint — works but rate-limited |
| MetaMask "wrong network" | The UI prompts to switch — no manual action needed |
| Stale CCTP attestation | The polling hook retries every 15 s; also try refreshing the History page |
| `tsc` errors after adding a lib | Run `pnpm run typecheck:libs` first to rebuild declaration files |
