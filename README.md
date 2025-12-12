# Tezos Bank dApp

A modern decentralized banking application built on the Tezos blockchain. Deposit and withdraw Tez (XTZ) through an intuitive web interface.

## Features

- 🔐 **Wallet Integration** - Connect your Tezos wallet using Beacon Wallet
- 💰 **Deposit** - Deposit Tez to the smart contract (1-100 XTZ)
- 💸 **Withdraw** - Withdraw your deposited Tez
- 📊 **Balance Display** - View your wallet balance in real-time
- 🎨 **Modern UI** - Beautiful, responsive interface with smooth animations

## Tech Stack

- **Frontend**: Svelte 5 + Vite
- **Blockchain**: Tezos (Shadownet testnet)
- **Wallet**: Beacon Wallet (@taquito/beacon-wallet)
- **SDK**: Taquito (@taquito/taquito)

## Getting Started

### Prerequisites

- Node.js (v16 or higher)
- npm or yarn

### Installation

```bash
npm install
```

### Development

```bash
npm run dev
```

The app will be available at `http://localhost:4000`

### Build

```bash
npm run build
```

## Usage

1. **Connect Wallet**: Click "Connect Wallet" and approve the connection in your wallet
2. **Get Testnet Tez**: Visit the [Shadownet Faucet](https://faucet.shadownet.teztnets.com/) to get testnet tokens
3. **Deposit**: Enter an amount (1-100 XTZ) and click "Deposit"
4. **Withdraw**: Click "Withdraw" to retrieve your deposited Tez

## Network

- **Network**: Shadownet Testnet
- **RPC URL**: `https://rpc.shadownet.teztnets.com`
- **Contract**: `KT1HtZfNKVcgPYCTuVPKm3cjEVAC4CKYrfjX`
- **Explorer**: [Shadownet TzKT](https://shadownet.tzkt.io/)

## License

MIT

