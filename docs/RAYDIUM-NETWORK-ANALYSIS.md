# Raydium Liquidity Pools - Network Fetching & Posting Analysis

This document analyzes how [Raydium's liquidity pools page](https://raydium.io/liquidity-pools/) fetches and posts data, including all API endpoints, request/response structures, and data flow patterns.

## Overview

Raydium's liquidity pools page uses a combination of REST API endpoints and Solana RPC calls to fetch pool data, statistics, and interact with the blockchain. The page primarily **fetches data** (GET requests) and uses **POST requests** for Solana RPC calls to interact with the blockchain.

## Key API Endpoints

### Base API URL
- **Main API**: `https://api-v3.raydium.io/`
- **RPC Endpoint**: `https://raydium-frontend.rpcpool.com/` (Solana RPC)
- **Image CDN**: `https://img-v1.raydium.io/icon/`
- **Third-party**: `https://lite-api.jup.ag/` (Jupiter token verification)

## Network Requests Breakdown

### 1. Initial Page Load Requests

#### GET Requests (Data Fetching)

**1. Mint/Token List**
```
GET https://api-v3.raydium.io/mint/list
```
- **Purpose**: Fetch list of all supported tokens/mints
- **Timing**: Called on page load (initial and subsequent)
- **Method**: XMLHttpRequest
- **Response**: List of token metadata (mint addresses, symbols, decimals, logos)

**2. Main Configuration Info**
```
GET https://api-v3.raydium.io/main/version
GET https://api-v3.raydium.io/main/auto-fee
GET https://api-v3.raydium.io/main/rpcs
GET https://api-v3.raydium.io/main/info
GET https://api-v3.raydium.io/main/mint-filter-config
GET https://api-v3.raydium.io/main/chain-time
```
- **Purpose**: Fetch system configuration, version, fee settings, RPC endpoints, and chain time
- **Timing**: Initial page load
- **Method**: XMLHttpRequest
- **Response**: Configuration data

**3. Pool List (Main Data)**
```
GET https://api-v3.raydium.io/pools/info/list-v2?sortType=desc&size=100
```
- **Purpose**: Fetch liquidity pool list with pagination
- **Query Parameters**:
  - `sortType`: Sort order (`desc` or `asc`)
  - `size`: Number of pools to return (default: 100)
  - `pageId`: Pagination cursor (optional)
- **Timing**: Initial page load, updated periodically
- **Method**: XMLHttpRequest
- **Response**: Array of pool objects with detailed information

**4. Token Prices**
```
GET https://api-v3.raydium.io/mint/price?mints={mint1},{mint2},{mint3}...
```
- **Purpose**: Fetch current prices for multiple tokens
- **Query Parameters**:
  - `mints`: Comma-separated list of mint addresses
- **Timing**: Called periodically to update prices
- **Method**: XMLHttpRequest
- **Response**: Price data for requested mints

**5. Main Info (TVL & Volume)**
```
GET https://api-v3.raydium.io/main/info
```
- **Purpose**: Fetch overall platform statistics (TVL, 24h volume)
- **Timing**: Initial page load, updated periodically
- **Response Example**:
```json
{
  "id": "5fd6b8dc-9eb3-4ab0-8f8e-e17d0fde7f25",
  "success": true,
  "data": {
    "volume24": 1005446139.4186554,
    "tvl": 2342117210.331859
  }
}
```

#### POST Requests (Solana RPC)

**1. Solana RPC Calls**
```
POST https://raydium-frontend.rpcpool.com/
```
- **Purpose**: Solana blockchain RPC calls
- **Method**: POST (JSON-RPC 2.0)
- **Content-Type**: `application/json`
- **Authentication**: Required (403 if not authenticated)
- **Request Body Format**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getHealth",
  "params": []
}
```
- **Common RPC Methods**:
  - `getHealth`: Check RPC node health
  - `getAccountInfo`: Get account information
  - `getProgramAccounts`: Get program accounts
  - `simulateTransaction`: Simulate transaction
  - `sendTransaction`: Submit transaction

### 2. Third-Party API Calls

**1. Jupiter Token Verification**
```
GET https://lite-api.jup.ag/tokens/v2/tag?query=verified
```
- **Purpose**: Fetch verified token list from Jupiter
- **Timing**: Initial page load
- **Method**: XMLHttpRequest
- **Response**: List of verified tokens

### 3. Asset/Image Requests

**1. Token Icons**
```
GET https://img-v1.raydium.io/icon/{mintAddress}.png
GET https://wsrv.nl/?fit=cover&w=20&h=20&url=https://img-v1.raydium.io/icon/{mintAddress}.png
```
- **Purpose**: Fetch token icons/logos
- **CDN**: Uses image proxy service (wsrv.nl) for optimized images
- **Timing**: Lazy loaded as pools are displayed

## API Response Structures

### Pool List Response (`/pools/info/list-v2`)

**Response Structure**:
```json
{
  "id": "request-id",
  "success": true,
  "data": {
    "data": [
      {
        "type": "Concentrated" | "Standard",
        "programId": "program-address",
        "id": "pool-id",
        "mintA": {
          "chainId": 101,
          "address": "mint-address",
          "programId": "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",
          "logoURI": "https://img-v1.raydium.io/icon/...",
          "symbol": "WSOL",
          "name": "Wrapped SOL",
          "decimals": 9,
          "tags": [],
          "extensions": {}
        },
        "mintB": { /* same structure as mintA */ },
        "price": 167.64896883999654,
        "mintAmountA": 101628.554746141,
        "mintAmountB": 2635818.643424,
        "feeRate": 0.0004,
        "openTime": "1723037622",
        "tvl": 19642475.33,
        "day": {
          "volume": 149559270.7089016,
          "volumeQuote": 149654367.14899614,
          "volumeFee": 59823.70828356064,
          "apr": 111.17,
          "feeApr": 111.17,
          "priceMin": 162.33766233766235,
          "priceMax": 189.13787437234944,
          "rewardApr": [0]
        },
        "week": { /* same structure as day */ },
        "month": { /* same structure as day */ },
        "pooltype": ["Clmm"],
        "farmUpcomingCount": 0,
        "farmOngoingCount": 0,
        "farmFinishedCount": 1,
        "config": {
          "id": "config-id",
          "index": 8,
          "protocolFeeRate": 120000,
          "tradeFeeRate": 400,
          "tickSpacing": 1,
          "fundFeeRate": 40000,
          "defaultRange": 0.1,
          "defaultRangePoint": [0.01, 0.05, 0.1, 0.2, 0.5]
        },
        "burnPercent": 40.92,
        "launchMigratePool": false,
        "rewardDefaultInfos": [
          {
            "mint": { /* token info */ },
            "perSecond": "1653",
            "startTime": "1756722600",
            "endTime": "1759746600"
          }
        ]
      }
    ],
    "nextPageId": "pagination-cursor"
  }
}
```

**Key Fields**:
- `type`: Pool type (`Concentrated` for CLMM, `Standard` for AMM)
- `id`: Unique pool identifier
- `mintA` / `mintB`: Token pair information
- `tvl`: Total Value Locked in USD
- `day/week/month`: Volume, fees, and APR data
- `price`: Current price (mintA/mintB)
- `feeRate`: Trading fee rate (e.g., 0.0004 = 0.04%)
- `rewardDefaultInfos`: Reward token information and rates

### Main Info Response (`/main/info`)

**Response Structure**:
```json
{
  "id": "request-id",
  "success": true,
  "data": {
    "volume24": 1005446139.4186554,
    "tvl": 2342117210.331859
  }
}
```

**Key Fields**:
- `volume24`: 24-hour trading volume in USD
- `tvl`: Total Value Locked across all pools in USD

## Data Fetching Flow

### Initial Page Load Sequence

```
1. Page HTML Loads
   ↓
2. Next.js App Initializes
   ↓
3. Initial API Calls (Parallel):
   - GET /mint/list (token list)
   - GET /main/version (version info)
   - GET /main/auto-fee (fee config)
   - GET /main/rpcs (RPC endpoints)
   ↓
4. Pool Data Fetching:
   - GET /pools/info/list-v2?sortType=desc&size=100
   - GET /main/info (TVL & volume)
   - GET /main/mint-filter-config (filter config)
   ↓
5. Token Price Updates:
   - GET /mint/price?mints=... (periodic updates)
   ↓
6. Asset Loading:
   - Token icons (lazy loaded)
   - Images (lazy loaded)
```

### Real-Time Updates

**Polling Pattern**:
- Pool list: Updated periodically (likely every 30-60 seconds)
- Token prices: Updated more frequently (every 5-10 seconds)
- TVL/Volume: Updated periodically with pool list

## POST Request Patterns

### Solana RPC POST Requests

**When Used**:
- User interactions (deposit, withdraw, swap)
- Transaction simulation
- Account balance checks
- Transaction submission

**Request Format**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "methodName",
  "params": [/* method-specific parameters */]
}
```

**Common RPC Methods**:

1. **Get Account Info**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getAccountInfo",
  "params": [
    "account-address",
    {
      "encoding": "jsonParsed"
    }
  ]
}
```

2. **Get Program Accounts**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "getProgramAccounts",
  "params": [
    "program-address",
    {
      "filters": [...],
      "encoding": "jsonParsed"
    }
  ]
}
```

3. **Simulate Transaction**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "simulateTransaction",
  "params": [
    "base64-encoded-transaction",
    {
      "encoding": "base64",
      "sigVerify": false,
      "replaceRecentBlockhash": true
    }
  ]
}
```

4. **Send Transaction**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "sendTransaction",
  "params": [
    "base64-encoded-transaction",
    {
      "encoding": "base64",
      "skipPreflight": false,
      "preflightCommitment": "confirmed"
    }
  ]
}
```

## Data Flow for User Actions

### Deposit Liquidity Flow

```
1. User Clicks "Deposit" Button
   ↓
2. Frontend Prepares Transaction
   - Validates user input
   - Calculates amounts
   - Builds transaction instruction
   ↓
3. POST /raydium-frontend.rpcpool.com/
   - Method: simulateTransaction
   - Simulates transaction to check if it will succeed
   ↓
4. User Approves Transaction
   - Wallet prompts for signature
   ↓
5. POST /raydium-frontend.rpcpool.com/
   - Method: sendTransaction
   - Submits signed transaction to blockchain
   ↓
6. Transaction Confirmation
   - Polls RPC for transaction status
   - Updates UI on confirmation
   ↓
7. Refresh Pool Data
   - GET /pools/info/list-v2 (updates pool TVL)
   - GET /main/info (updates total TVL)
```

### Viewing Pool Details Flow

```
1. User Clicks on Pool
   ↓
2. Navigation to Pool Detail Page
   - Route: /liquidity-pools/{poolId}
   ↓
3. Fetch Pool Details
   - GET /pools/info/{poolId} (if available)
   - Or filter from existing pool list
   ↓
4. Fetch Additional Data
   - GET /mint/price?mints=... (for pool tokens)
   - RPC calls for user's position (if wallet connected)
   ↓
5. Display Pool Information
   - Show pool statistics
   - Show user's position (if applicable)
```

## Network Request Patterns

### Request Headers

**GET Requests**:
```
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Cache-Control: no-cache
Pragma: no-cache
Referer: https://raydium.io/liquidity-pools/
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

## Pagination

**Pool List Pagination**:
- Uses cursor-based pagination
- Response includes `nextPageId` field
- Subsequent requests use `pageId` query parameter:
  ```
  GET /pools/info/list-v2?sortType=desc&size=100&pageId={nextPageId}
  ```

## Error Handling

**API Error Responses**:
```json
{
  "id": "request-id",
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Error message"
  }
}
```

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

## Authentication

**Public Endpoints** (No Authentication):
- `/mint/list`
- `/pools/info/list-v2`
- `/main/info`
- `/main/version`
- `/mint/price`

**RPC Endpoint** (Authentication Required):
- `/raydium-frontend.rpcpool.com/` - Returns 403 if not authenticated
- Likely requires API key or origin validation

## Caching Strategy

**Cached Resources**:
- Token icons: Cached by browser/CDN
- Static assets: Cached with long TTL
- API responses: May use cache headers for immutable data

**Non-Cached Resources**:
- Pool list: Fetched fresh (real-time data)
- Prices: Updated frequently
- TVL/Volume: Updated periodically

## Rate Limiting

**Observation**:
- Multiple parallel requests on page load
- Periodic polling for price updates
- No apparent rate limiting on public endpoints
- RPC endpoint may have rate limits (403 suggests authentication/rate limiting)

## Request Timing

**Initial Load**:
- Pool list: ~138ms
- Main info: ~111ms
- Mint list: ~190ms
- Token prices: ~124ms

**Periodic Updates**:
- Prices: Updated every few seconds
- Pool list: Updated every 30-60 seconds (estimated)
- TVL/Volume: Updated with pool list

## Example: Complete Data Fetching Sequence

```
Page Load (t=0ms)
├── GET /mint/list (190ms)
├── GET /main/version (160ms)
├── GET /main/auto-fee (176ms)
└── GET /main/rpcs (159ms)

After Initial Config (t=200ms)
├── GET /pools/info/list-v2?sortType=desc&size=100 (138ms)
├── GET /main/info (111ms)
└── GET /main/mint-filter-config (106ms)

Token Assets (t=400ms)
├── GET /mint/price?mints=... (124ms)
└── Multiple icon/image requests (lazy loaded)

Periodic Updates (every 5-30 seconds)
├── GET /mint/price?mints=... (price updates)
└── GET /pools/info/list-v2?sortType=desc&size=100 (pool updates)
```

## Key Differences from Orderly Network

### Data Fetching

| Aspect | Raydium | Orderly Network |
|--------|---------|-----------------|
| **API Base** | `api-v3.raydium.io` | `api.orderly.org` |
| **Blockchain** | Solana (native) | EVM + Solana |
| **RPC Calls** | Solana RPC (JSON-RPC 2.0) | Web3 providers |
| **Data Format** | Custom JSON API | RESTful API |
| **Authentication** | RPC endpoint requires auth | API key + signature |

### Pool Data Structure

**Raydium**:
- Pool types: `Concentrated` (CLMM), `Standard` (AMM)
- Pool data includes: `mintA`, `mintB`, `tvl`, `day/week/month` stats
- Reward information embedded in pool data

**Orderly Network**:
- Focus on perpetual futures, not liquidity pools
- Uses different data structures for trading pairs

## Summary

Raydium's liquidity pools page uses:

1. **REST API** (`api-v3.raydium.io`) for:
   - Pool list data
   - Token metadata
   - Price information
   - Platform statistics (TVL, volume)
   - Configuration data

2. **Solana RPC** (`raydium-frontend.rpcpool.com`) for:
   - Blockchain interactions
   - Transaction simulation
   - Transaction submission
   - Account queries

3. **Third-Party APIs** for:
   - Token verification (Jupiter)
   - Image optimization (CDN services)

4. **Data Flow**:
   - **GET requests**: Fetch pool data, prices, statistics
   - **POST requests**: Solana RPC calls for blockchain interactions
   - **Polling**: Periodic updates for prices and pool data
   - **Lazy loading**: Images and icons loaded on demand

The page primarily **fetches data** (GET) and uses **POST requests** only for Solana RPC calls to interact with the blockchain. All pool data is retrieved via REST API endpoints, while user transactions are handled through Solana RPC POST requests.

