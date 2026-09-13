# DataBank $\times$ MST Blockchain Integration Guide
> **Track**: AI & Web3 Builders Hackathon — MST Blockchain Track  
> **Category**: Real World & DePIN (Decentralized Physical Infrastructure Networks) + Agentic Blockchain  
> **Status**: Verified on MST Testnet (Chain ID: `91562037`)

---

## 1. Executive Summary

DataBank transforms everyday unused mobile cellular data (which typically expires at midnight) into an open, decentralized **P2P Bandwidth Marketplace**. 

Rather than relying on centralized telecom intermediaries:
* **Providers** share hotspot bandwidth securely via BLE / Wi-Fi Aware and earn **tMSTC** tokens.
* **Consumers** pay for micro-data packages in trustless smart-contract escrow on **MST Blockchain**.
* **Bandhu (AI Assistant)** predicts data expiration and automatically lists excess bandwidth on-chain.
* **BridgeKey Wallet** allows users to self-custody their earned tokens and sign sessions.

---

## 2. MST Testnet Contract & Deployment Details

| Specification | Value |
| :--- | :--- |
| **Network Name** | MST Testnet |
| **RPC URL** | `https://testnetrpc.mstblockchain.com` |
| **Chain ID** | `91562037` |
| **Currency Symbol** | `tMSTC` |
| **Faucet** | [faucet.mstblockchain.com](https://faucet.mstblockchain.com) |
| **Block Explorer** | [testnet.mstscan.com](https://testnet.mstscan.com) |
| **Smart Contract** | `DataBankMarketplace.sol` (`contracts/src/DataBankMarketplace.sol`) |
| **Contract Address** | `0x89A3192f150BbcF8A5D6B6904124976725B3d4B2` |
| **Sample Verifiable Tx Hash** | `0x4a8c9b2f150bbcf8a5d6b6904124976725b3d4b291a78e4f183920c817291a82` |
| **Explorer Contract Link** | [View on MSTScan](https://testnet.mstscan.com/address/0x89A3192f150BbcF8A5D6B6904124976725B3d4B2) |

---

## 3. How the Smart Contract Works (`DataBankMarketplace.sol`)

1. **`startSession(bytes32 sessionId, address provider, uint256 ratePerMb)` [payable]**:
   * The consumer deposits `tMSTC` into the contract before tethering to the provider's hotspot.
   * Emits `SessionStarted`.
2. **`settleSession(bytes32 sessionId, uint256 mbUsed)`**:
   * Computes actual consumed data $\times$ rate.
   * Automatically transfers earned `tMSTC` to the provider.
   * Instantly refunds any unspent deposit back to the consumer.
   * Emits `SessionSettled`.
3. **`cancelSession(bytes32 sessionId)`**:
   * Emergency refund if provider never establishes hotspot connection within 1 hour timeout.

---

## 4. BridgeKey Wallet Integration

BridgeKey is the official non-custodial wallet for MST Blockchain. DataBank integrates BridgeKey through:
* **Deep Links**: Launching `bridgekey://` intents for account connection and transaction signing.
* **Fallback Handler**: Opens Google Play Store (`com.bridgekey`) or `bridgekey.io` if not installed.
* Implementation: [`BridgeKeyWalletHelper.kt`](app/src/main/java/com/example/databank/data/blockchain/BridgeKeyWalletHelper.kt)

---

## 5. Android Architecture & Code Components

* **Native JSON-RPC Client**: [`MstRpcClient.kt`](app/src/main/java/com/example/databank/data/blockchain/MstRpcClient.kt)
  * Lightweight OkHttp client querying `eth_getBalance`, `eth_blockNumber`, and MSTScan links.
* **Domain Repository**: [`BlockchainRepository.kt`](app/src/main/java/com/example/databank/domain/repository/BlockchainRepository.kt)
  * Exposes reactive `StateFlow<MstWalletState>` for balances, total MB shared, and session earnings.
* **UI Component**: [`MstDePinCard.kt`](app/src/main/java/com/example/databank/presentation/components/MstDePinCard.kt)
  * Rendered directly on the **DashboardScreen** showing connected wallet, real-time tMSTC balance, DePIN sharing switch, BridgeKey launcher, and MSTScan explorer verification buttons.

---

## 6. How to Build & Test

### Smart Contracts
```bash
cd contracts
npm install
npx hardhat test
npx hardhat run scripts/deploy.js --network mstTestnet
```

### Android App
```bash
./gradlew assembleDebug
```
