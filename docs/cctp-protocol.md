# CCTP — Cross-Chain Transfer Protocol

## What is CCTP?

Circle's Cross-Chain Transfer Protocol is a permissionless on-chain utility that lets USDC move **natively** across blockchains. Instead of locking tokens in a bridge contract and issuing wrapped copies, CCTP:

1. **Burns** USDC on the source chain via Circle's `TokenMessenger` contract
2. **Attests** the burn event through Circle's off-chain Iris API
3. **Mints** fresh USDC on the destination chain via the destination `MessageTransmitter`

The result: **no wrapped tokens, no custodian, no synthetic assets** — the USDC on the destination is canonical Circle USDC.

---

## How Arc Bridge Uses CCTP

Arc Bridge wraps Circle's **App Kit SDK** (`@circle-fin/app-kit`) with a custom viem adapter (`@circle-fin/adapter-viem-v2`).

```ts
const kit = new AppKit();
const adapter = await createViemAdapterFromProvider({ provider: proxiedWallet });

const result = await kit.bridge({
  from: { adapter, chain: fromConfig.bridgeChain },
  to:   { adapter, chain: toConfig.bridgeChain   },
  amount: bridgeAmount,   // post-fee USDC (decimal string)
});
```

The SDK returns a `result` object with a `steps` array that updates in place as each stage progresses.

---

## Step-by-Step Flow

### 1. Fee Transfer (Arc Bridge custom step)
Before invoking the SDK, Arc Bridge collects the protocol fee via a plain ERC-20 `transfer(treasury, fee)`. This step is not part of CCTP — it is Arc Bridge's own revenue mechanism.

### 2. Approve
`USDC.approve(tokenMessengerAddress, bridgeAmount)`

Authorises Circle's `TokenMessenger` contract to pull USDC from the user's wallet.

### 3. Burn
`TokenMessenger.depositForBurn(amount, destinationDomain, mintRecipient, burnToken)`

Burns the USDC. The `MessageTransmitter` emits a `MessageSent` event with an encoded message.

### 4. FetchAttestation
The SDK calls Circle's Iris API with the keccak256 hash of the `MessageSent` message body. Once Circle's attesters sign it, the API returns a signed attestation.

- Sandbox (testnet): typically 10–30 seconds
- Mainnet: typically 2–5 seconds

### 5. Mint
`MessageTransmitter.receiveMessage(message, attestation)`

The signed attestation is submitted on the destination chain. The `TokenMessenger` mints the equivalent USDC to the recipient.

---

## Domain IDs

| Network | CCTP Domain |
|---------|-------------|
| Ethereum (Sepolia) | 0 |
| Avalanche (Fuji) | 1 |
| Optimism (Sepolia) | 2 |
| Arbitrum (Sepolia) | 3 |
| Base (Sepolia) | 6 |
| Arc Testnet | 9 |

---

## Attestation Polling

Arc Bridge implements its own attestation poller in `src/lib/hooks/use-cctp-polling.ts`. It:
- Reads all `pending` transactions from the Zustand store every **15 seconds**
- Extracts the `MessageSent` hash from the burn receipt
- Polls the Iris sandbox API: `https://iris-api-sandbox.circle.com/v1/attestations/{hash}`
- Updates the transaction status to `completed` and saves the `mintTxHash` once confirmed

This means transaction status continues to update **even if the user navigates away** from the bridge page.

---

## References

- [Circle CCTP Docs](https://developers.circle.com/stablecoins/docs/cctp-getting-started)
- [Circle App Kit](https://developers.circle.com/stablecoins/docs/cctp-app-kit)
- [Iris API](https://iris-api-sandbox.circle.com)
