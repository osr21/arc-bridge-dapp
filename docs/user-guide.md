# User Guide

## Prerequisites

- A browser with MetaMask (or any injected EVM wallet) installed
- Testnet USDC on a supported source chain
- A small amount of that chain's gas token (ETH on Sepolia, etc.)

## Getting Testnet USDC

Visit **https://faucet.circle.com** and request USDC for any supported testnet. You will need to paste your wallet address.

For Arc Testnet gas (USDC): bridge some USDC from Sepolia to Arc, or use the same Circle faucet with Arc selected.

---

## Bridging USDC

### Step 1 — Connect your wallet

Click **Connect Wallet** in the top-right. Select MetaMask or any RainbowKit-supported wallet.

### Step 2 — Select direction

The default direction is **Sepolia → Arc Testnet**. Use the dropdown on the "From" network to choose a different source chain (Base Sepolia, Arb Sepolia, OP Sepolia). Click the **⇅ swap** button to reverse direction.

### Step 3 — Enter amount

Type the USDC amount (minimum **1 USDC**). The fee breakdown appears below:

| Line | Description |
|------|-------------|
| Protocol fee | 0.30% of the amount, min $0.25 USDC |
| You receive | amount − fee, delivered on the destination chain |

### Step 4 — Bridge

Click **Bridge USDC**. You will be prompted to sign **up to 3 transactions** in MetaMask:

1. **Fee Transfer** — sends the protocol fee to the treasury on the source chain
2. **Approve** — authorises Circle's burn contract to spend your USDC
3. **Burn** — burns USDC on the source chain

After the burn confirms, Circle's attestation service processes the message (~10–20 seconds on testnet). The **Mint** step then executes automatically on the destination chain — no additional wallet signature needed.

### Step 5 — Success screen

After the mint confirms you will see:
- 🎉 **Bridge Complete!** with the bridged amount
- Links to both the burn and mint transactions on their respective explorers
- A **Share** button that copies a tweet-ready summary
- A **History** button to view all past bridges

---

## Gas Dashboard

Navigate to **Gas** in the top menu.

- **Your Gas Balance** — your USDC balance on Arc Testnet (connect wallet to view)
- **Network Status** — live Arc gas price (typically ~20 Gwei, ~$0.01/tx) and network load
- **Get Gas** — one-click copy of your wallet address plus a link to the Circle faucet

---

## Transaction History

Navigate to **History** in the top menu.

Each completed USDC bridge shows:
- Amount, direction, timestamp, status badge
- 🔗 **Burn tx** link (source chain explorer)
- 🟢 **Mint tx** link (Arc explorer) — appears once the mint confirms

Pending bridges show an **Attesting** badge while Circle processes the attestation.

---

## NFT Bridge

> The NFT Bridge UI demonstrates the lock-and-mint pattern. Actual bridging requires deploying your own ERC-721 bridge contract (CCTP only transports USDC natively).

Navigate to **NFT Bridge** and configure your contract address and Thirdweb credentials in **Settings** to enable real minting.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| "Insufficient balance" | Ensure you have enough USDC on the *source* chain, not the destination |
| Transaction stuck on "Attesting" | Normal on testnet — can take up to 2 minutes. The History page polls automatically. |
| MetaMask shows wrong network | The app will prompt you to switch — click the Switch button in the banner |
| Mint never arrives | Check the Circle Iris API status at https://iris-api-sandbox.circle.com |
