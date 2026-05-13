# Protocol Fee Model

## Overview

Arc Bridge charges a small protocol fee on every USDC bridge transaction. The fee is collected as a real on-chain ERC-20 transfer **before** Circle's CCTP flow begins, ensuring it is settled unconditionally.

---

## Fee Schedule

| Transaction type | Rate | Minimum |
|------------------|------|---------|
| USDC Bridge | 0.30% (30 bps) | $0.25 USDC |
| NFT Bridge | $0.50 flat | — |

### Examples

| Bridge amount | Fee | You receive |
|---------------|-----|-------------|
| 1 USDC | $0.25 (min) | 0.75 USDC |
| 10 USDC | $0.25 (min) | 9.75 USDC |
| 100 USDC | $0.30 | 99.70 USDC |
| 1,000 USDC | $3.00 | 997.00 USDC |

---

## On-Chain Collection

```
calcProtocolFee(amount)
  → fee = max(amount × 0.003, 0.25)
  → bridgeAmount = amount − fee

sendFeeTransfer(walletClient, fromChainId, fee)
  → USDC.transfer(TREASURY_ADDRESS, fee)   // on source chain
  → returns feeTxHash

kit.bridge({ amount: bridgeAmount })        // Circle CCTP with post-fee amount
```

The fee transfer and bridge are two separate transactions. If the user rejects the fee transfer, the CCTP bridge never starts — preventing a situation where the fee is unpaid but the bridge proceeds.

---

## Treasury

All fees are accumulated at:

```
0xdb5019b8DfbccEF8906C39B16a4870082eAbBc4C
```

This is a plain EOA. Fees land on the **source chain** (the chain the user bridges from), so the treasury may accumulate USDC across Sepolia, Base Sepolia, Arbitrum Sepolia, Optimism Sepolia, and Arc Testnet.

The Settings page in the dApp displays a running total of estimated fees based on your local transaction history.

---

## Implementation Reference

| File | Responsibility |
|------|---------------|
| `src/lib/constants.ts` | `PROTOCOL_FEE_BPS`, `PROTOCOL_FEE_MIN_USDC`, `TREASURY_ADDRESS`, `calcProtocolFee()` |
| `src/lib/fee.ts` | `sendFeeTransfer()`, `feeExplorerUrl()` |
| `src/pages/home.tsx` | Invokes fee transfer, passes `bridgeAmount` to Circle SDK |
| `src/pages/settings.tsx` | Displays cumulative fee estimates from history |
