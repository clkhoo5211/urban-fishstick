# Raydium Bridge (Wormhole Connect) - Network Fetching & Posting Analysis

This document analyzes how [Raydium's bridge page](https://raydium.io/bridge/) fetches and posts data, including all API endpoints, request/response structures, and data flow patterns.

## Overview

Raydium's bridge page uses **Wormhole Connect**, a third-party cross-chain bridging widget that enables users to transfer assets between different blockchains (e.g., Ethereum, Solana, BSC, etc.). The page is powered by [Wormhole's Connect product](https://wormhole.com/products/connect), which handles the cross-chain bridging logic.

## Key API Endpoints

### Base API URLs
- **Wormhole API**: `https://api.wormhole.com/` (cross-chain bridge API)
- **Main Raydium API**: `https://api-v3.raydium.io/` (shared config)
- **Ethereum RPC**: `https://ethereum-rpc.publicnode.com/` (Ethereum blockchain)
- **Solana RPC**: `https://raydium-frontend.rpcpool.com/` (Solana blockchain)
- **Wormhole Connect**: Embedded widget/iframe

## Network Requests Breakdown

### 1. Initial Page Load Requests

#### GET Requests (Data Fetching)

**1. Shared Configuration (Main API)**
```
GET https://api-v3.raydium.io/mint/list
GET https://api-v3.raydium.io/main/version
GET https://api-v3.raydium.io/main/auto-fee
GET https://api-v3.raydium.io/main/rpcs
GET https://api-v3.raydium.io/mint/price?mints=...
GET https://api-v3.raydium.io/main/chain-time
```
- **Purpose**: Shared configuration and token metadata (same as other pages)
- **Timing**: Initial page load
- **Response**: Standard Raydium API responses

**2. Third-Party Token Verification**
```
GET https://lite-api.jup.ag/tokens/v2/tag?query=verified
```
- **Purpose**: Fetch verified token list from Jupiter
- **Timing**: Initial page load
- **Response**: List of verified tokens

#### POST Requests (Blockchain RPC)

**1. Ethereum RPC Calls**
```
POST https://ethereum-rpc.publicnode.com/
```
- **Purpose**: Ethereum blockchain interactions
- **Method**: POST (JSON-RPC 2.0)
- **Content-Type**: `application/json`
- **Timing**: When Ethereum wallet is connected or when fetching Ethereum chain data
- **Common Methods**:
  - `eth_chainId`: Get chain ID
  - `eth_getBalance`: Get account balance
  - `eth_call`: Call contract methods
  - `eth_sendTransaction`: Send transaction
  - `eth_getTransactionReceipt`: Get transaction receipt

**2. Solana RPC Calls**
```
POST https://raydium-frontend.rpcpool.com/
```
- **Purpose**: Solana blockchain interactions
- **Method**: POST (JSON-RPC 2.0)
- **Content-Type**: `application/json`
- **Timing**: When Solana wallet is connected or when fetching Solana chain data
- **Common Methods**:
  - `getHealth`: Check RPC node health
  - `getAccountInfo`: Get account information
  - `getBalance`: Get account balance
  - `sendTransaction`: Submit transaction
  - `getTransaction`: Get transaction status

**3. Wormhole API Calls** (Likely)
```
POST https://api.wormhole.com/v1/quote
POST https://api.wormhole.com/v1/transfer
POST https://api.wormhole.com/v1/status
```
- **Purpose**: Cross-chain bridge operations
- **Method**: POST (likely REST API or JSON-RPC)
- **Note**: These calls may be made from within the Wormhole Connect widget/iframe, so they may not appear in the main page's network tab

### 2. Wormhole Connect Widget

**Wormhole Connect** is an embedded widget that handles:
- Chain selection (source and destination)
- Token selection
- Amount input
- Quote fetching
- Transaction execution
- Bridge status tracking

**Widget Integration**:
- Likely embedded as an iframe or React component
- Handles its own API calls internally
- May use WebSocket connections for real-time updates
- Communicates with parent page via postMessage API

## Data Fetching Flow

### Initial Page Load Sequence

```
1. Page HTML Loads
   ↓
2. Next.js App Initializes
   ↓
3. Initial API Calls (Parallel):
   - GET /mint/list (token list from main API)
   - GET /main/version (version info)
   - GET /main/auto-fee (fee config)
   - GET /main/rpcs (RPC endpoints)
   ↓
4. Wormhole Connect Widget Initializes:
   - Loads widget JavaScript
   - Connects to Wormhole API
   - Fetches supported chains
   - Fetches supported tokens
   ↓
5. RPC Endpoint Detection:
   - POST /ethereum-rpc.publicnode.com/ (if Ethereum chain needed)
   - POST /raydium-frontend.rpcpool.com/ (if Solana chain needed)
   ↓
6. Price Updates:
   - GET /mint/price?mints=... (periodic updates)
```

### Bridge Transaction Flow

```
1. User Connects Source Wallet
   ↓
2. User Selects Source Chain & Token
   ↓
3. Widget Fetches Quote:
   - POST /api.wormhole.com/v1/quote (likely inside widget)
   - Parameters: sourceChain, destinationChain, sourceToken, amount
   ↓
4. User Enters Amount
   ↓
5. Widget Calculates Destination Amount:
   - Uses quote from Wormhole API
   - Displays fees and estimated time
   ↓
6. User Connects Destination Wallet (if different)
   ↓
7. User Approves Transaction
   ↓
8. Source Chain Transaction:
   - POST /ethereum-rpc.publicnode.com/ or /raydium-frontend.rpcpool.com/
   - Method: sendTransaction
   - Submits lock/burn transaction on source chain
   ↓
9. Wormhole Relayer Processing:
   - Wormhole validators observe source chain
   - Generate VAA (Verifiable Action Approval)
   - Relay to destination chain
   ↓
10. Destination Chain Transaction:
    - POST /ethereum-rpc.publicnode.com/ or /raydium-frontend.rpcpool.com/
    - Method: sendTransaction
    - Submits redeem/mint transaction on destination chain
    ↓
11. Transaction Confirmation:
    - Polls RPC for transaction status
    - Updates UI on confirmation
```

## Wormhole API Endpoints (Inferred)

Based on Wormhole's architecture and typical bridge implementations:

### Quote Endpoint
```
POST https://api.wormhole.com/v1/quote
```
**Request Body**:
```json
{
  "sourceChain": 1,
  "destinationChain": 5,
  "sourceToken": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "destinationToken": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
  "amount": "1000000000",
  "slippageTolerance": "0.01"
}
```

**Response**:
```json
{
  "sourceAmount": "1000000000",
  "destinationAmount": "999000000",
  "fee": "1000000",
  "estimatedTime": 300,
  "route": {
    "steps": [...]
  }
}
```

### Transfer Endpoint
```
POST https://api.wormhole.com/v1/transfer
```
**Request Body**:
```json
{
  "sourceChain": 1,
  "destinationChain": 5,
  "sourceToken": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "destinationToken": "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v",
  "amount": "1000000000",
  "sourceAddress": "0x...",
  "destinationAddress": "..."
}
```

### Status Endpoint
```
GET https://api.wormhole.com/v1/status/{transactionHash}
```
**Response**:
```json
{
  "status": "pending" | "confirmed" | "completed" | "failed",
  "sourceTxHash": "0x...",
  "destinationTxHash": "...",
  "vaa": "...",
  "timestamp": 1234567890
}
```

## RPC Request Patterns

### Ethereum RPC Requests

**Get Chain ID**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_chainId",
  "params": []
}
```

**Get Balance**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_getBalance",
  "params": [
    "0x...",
    "latest"
  ]
}
```

**Send Transaction**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "eth_sendTransaction",
  "params": [{
    "from": "0x...",
    "to": "0x...",
    "value": "0x...",
    "data": "0x...",
    "gas": "0x...",
    "gasPrice": "0x..."
  }]
}
```

### Solana RPC Requests

**Get Balance**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getBalance",
  "params": ["account-address"]
}
```

**Send Transaction**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "sendTransaction",
  "params": [
    "base64-encoded-transaction",
    {
      "encoding": "base64",
      "skipPreflight": false
    }
  ]
}
```

## Network Request Patterns

### Request Headers

**GET Requests**:
```
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Cache-Control: no-cache
Pragma: no-cache
Referer: https://raydium.io/bridge/
```

**POST Requests (RPC)**:
```
Content-Type: application/json
Accept: application/json
```

### Response Headers

**API Responses**:
```
Content-Type: application/json
Access-Control-Allow-Origin: *
Cache-Control: public, max-age=...
```

## Supported Chains

Based on Wormhole's capabilities, the bridge likely supports:

**EVM Chains**:
- Ethereum (Mainnet)
- BNB Chain (BSC)
- Polygon
- Avalanche
- Arbitrum
- Optimism
- Base
- And more...

**Non-EVM Chains**:
- Solana
- Sui
- Aptos
- And more...

## Supported Tokens

Common bridged tokens include:
- USDC (Universal USD Coin)
- USDT (Tether)
- ETH (Ethereum)
- SOL (Solana)
- Wrapped tokens (WETH, WSOL, etc.)
- Native tokens on each chain

## Error Handling

**RPC Error Responses**:
```json
{
  "jsonrpc": "2.0",
  "error": {
    "code": -32600,
    "message": "Invalid Request"
  },
  "id": 1
}
```

**Wormhole API Error Responses** (Inferred):
```json
{
  "error": {
    "code": "INSUFFICIENT_BALANCE",
    "message": "Insufficient balance for transaction"
  }
}
```

## Authentication

**Public Endpoints** (No Authentication):
- Main Raydium API endpoints
- Public RPC endpoints (rate-limited)
- Wormhole API (may require API key for some endpoints)

**RPC Endpoints**:
- Public RPC endpoints may have rate limits
- Some RPC providers require authentication for higher rate limits

## Caching Strategy

**Cached Resources**:
- Token metadata: Cached by browser/CDN
- Chain configurations: Cached with long TTL
- Static assets: Cached with long TTL

**Non-Cached Resources**:
- Quotes: Fetched fresh (real-time pricing)
- Transaction status: Polled frequently
- Account balances: Updated on demand

## Rate Limiting

**Observation**:
- Multiple parallel RPC requests on page load
- Frequent polling for transaction status
- Real-time quote updates
- Public RPC endpoints may have rate limits
- Wormhole API may have rate limits

## Request Timing

**Initial Load**:
- Main API calls: ~160-190ms
- RPC health checks: ~100-200ms
- Widget initialization: Variable (depends on widget load time)

**Bridge Operations**:
- Quote fetching: ~500-1000ms (depends on network)
- Transaction submission: ~1000-3000ms (depends on chain)
- Transaction confirmation: 10-60 seconds (depends on chain)

## Example: Complete Bridge Transaction Sequence

```
Page Load (t=0ms)
├── GET /mint/list (190ms)
├── GET /main/version (160ms)
├── GET /main/auto-fee (176ms)
└── GET /main/rpcs (159ms)

Widget Initialization (t=200ms)
├── Wormhole Connect widget loads
├── Fetches supported chains
└── Fetches supported tokens

User Actions (t=1000ms+)
├── User connects source wallet
├── POST /ethereum-rpc.publicnode.com/ (getBalance)
├── User selects token and enters amount
├── POST /api.wormhole.com/v1/quote (quote request)
├── User approves transaction
├── POST /ethereum-rpc.publicnode.com/ (sendTransaction)
├── Transaction confirmation (polling)
└── Destination chain transaction (automatic via relayer)
```

## Key Differences from Other Pages

### Architecture

| Aspect | Bridge | Liquidity Pools | Launchpad |
|--------|--------|-----------------|-----------|
| **Primary Integration** | Wormhole Connect (3rd party widget) | Raydium API | Launchpad API |
| **Blockchain Support** | Multi-chain (EVM + Solana) | Solana only | Solana only |
| **RPC Endpoints** | Multiple (Ethereum + Solana) | Solana only | Solana only |
| **Transaction Type** | Cross-chain bridge | Pool operations | Token creation/trading |

### Data Flow

**Bridge**:
- Uses third-party widget (Wormhole Connect)
- Requires multiple RPC endpoints
- Involves relayers and validators
- Multi-step transaction process

**Other Pages**:
- Direct API integration
- Single blockchain (Solana)
- Direct transaction flow
- Simpler data fetching

## Security Considerations

**Bridge Security**:
- Wormhole uses a validator network for security
- VAAs (Verifiable Action Approvals) ensure transaction integrity
- Relayers are responsible for executing destination transactions
- Users should verify destination addresses

**Wallet Security**:
- Users must approve transactions on both chains
- Private keys never leave the wallet
- Transaction simulation before execution

## Summary

Raydium's bridge page uses:

1. **Wormhole Connect Widget** (Third-party integration) for:
   - Cross-chain bridge UI
   - Chain and token selection
   - Quote fetching
   - Transaction execution
   - Status tracking

2. **Main Raydium API** (`api-v3.raydium.io`) for:
   - Shared configuration
   - Token metadata
   - Price information

3. **Multiple RPC Endpoints** for:
   - Ethereum RPC (`ethereum-rpc.publicnode.com`)
   - Solana RPC (`raydium-frontend.rpcpool.com`)
   - Blockchain interactions on both chains

4. **Wormhole API** (likely) for:
   - Cross-chain bridge quotes
   - Transaction routing
   - Status tracking

5. **Data Flow**:
   - **GET requests**: Fetch configuration, token metadata, prices
   - **POST requests**: RPC calls for blockchain interactions, Wormhole API calls for bridge operations
   - **Widget-based**: Most bridge-specific API calls happen inside the Wormhole Connect widget
   - **Multi-chain**: Supports bridging between multiple blockchains

The page primarily uses a **third-party widget** (Wormhole Connect) that handles most of the bridge-specific API calls internally. The main page provides the wrapper and shared configuration, while the widget manages the actual cross-chain bridging operations.

