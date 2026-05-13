# Chain Configuration

## Arc Testnet

| Property | Value |
|----------|-------|
| Chain ID | 5042002 |
| Network name | Arc Testnet |
| RPC URL | https://rpc.testnet.arc.network |
| Block explorer | https://testnet.arcscan.app |
| Gas token | USDC (native, 18 decimals internally) |
| USDC contract | `0x3600000000000000000000000000000000000000` |
| Faucet | https://faucet.circle.com |

> **Note:** Arc's native gas token is USDC. The ERC-20 interface uses 6 decimals but the chain stores amounts internally with 18 decimals. Do not mix these — the bridge always works with the 6-decimal ERC-20 interface.

Arc Testnet is defined as a custom chain via viem's `defineChain` since it is not in wagmi's built-in chain list.

---

## Supported Source/Destination Chains

### Ethereum Sepolia
| Property | Value |
|----------|-------|
| Chain ID | 11155111 |
| RPC (Alchemy) | https://eth-sepolia.g.alchemy.com/v2/{key} |
| Explorer | https://sepolia.etherscan.io/tx/ |
| USDC | `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238` |
| CCTP Domain | 0 |

### Base Sepolia
| Property | Value |
|----------|-------|
| Chain ID | 84532 |
| RPC (Alchemy) | https://base-sepolia.g.alchemy.com/v2/{key} |
| Explorer | https://sepolia.basescan.org/tx/ |
| USDC | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` |
| CCTP Domain | 6 |

### Arbitrum Sepolia
| Property | Value |
|----------|-------|
| Chain ID | 421614 |
| RPC (Alchemy) | https://arb-sepolia.g.alchemy.com/v2/{key} |
| Explorer | https://sepolia.arbiscan.io/tx/ |
| USDC | `0x75faf114eafb1BDbe2F0316DF893fd58CE46AA4d` |
| CCTP Domain | 3 |

### Optimism Sepolia
| Property | Value |
|----------|-------|
| Chain ID | 11155420 |
| RPC (Alchemy) | https://opt-sepolia.g.alchemy.com/v2/{key} |
| Explorer | https://sepolia-optimism.etherscan.io/tx/ |
| USDC | `0x5fd84259d66Cd46123540766Be93DFE6D43130D7` |
| CCTP Domain | 2 |

---

## Adding MetaMask Network

To add Arc Testnet to MetaMask manually:

1. Open MetaMask → Settings → Networks → Add Network
2. Fill in:
   - **Network name:** Arc Testnet
   - **RPC URL:** https://rpc.testnet.arc.network
   - **Chain ID:** 5042002
   - **Currency symbol:** USDC
   - **Block explorer:** https://testnet.arcscan.app

The dApp calls `wallet_addEthereumChain` automatically when you select Arc as the destination and switch networks.
