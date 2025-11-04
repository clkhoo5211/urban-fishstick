# Raydium Launchpad (LaunchLab) - Network Fetching & Posting Analysis

This document analyzes how [Raydium's launchpad page](https://raydium.io/launchpad/) fetches and posts data, including all API endpoints, request/response structures, and data flow patterns.

## Overview

Raydium's LaunchLab (launchpad) page uses a **separate API service** (`launch-mint-v1.raydium.io`) specifically for token launch data, in addition to the main Raydium API (`api-v3.raydium.io`). The page displays newly launched tokens, trending tokens, and provides functionality for users to create and trade tokens.

## Key API Endpoints

### Base API URLs
- **Launchpad API**: `https://launch-mint-v1.raydium.io/`
- **Main Raydium API**: `https://api-v3.raydium.io/` (shared config)
- **Solana RPC**: `https://raydium-frontend.rpcpool.com/` (blockchain interactions)
- **Image CDN**: `https://img-v1.raydium.io/icon/`, IPFS gateways, various CDNs

## Network Requests Breakdown

### 1. Initial Page Load Requests

#### GET Requests (Data Fetching)

**1. Token Launch List (Main Data)**
```
GET https://launch-mint-v1.raydium.io/get/list?sort=hotToken&size=100&mintType=default&includeNsfw=false
GET https://launch-mint-v1.raydium.io/get/list?sort=hotToken&size=100&mintType=default&includeNsfw=false&platformId=PlatformWhiteList
```
- **Purpose**: Fetch list of launched tokens, sorted by popularity/heat
- **Query Parameters**:
  - `sort`: Sort order (`hotToken`, `new`, etc.)
  - `size`: Number of tokens to return (default: 100)
  - `mintType`: Token type filter (`default`)
  - `includeNsfw`: Include NSFW content (`false`)
  - `platformId`: Filter by platform (optional, e.g., `PlatformWhiteList`)
- **Timing**: Initial page load, updated periodically
- **Method**: XMLHttpRequest
- **Response**: Array of token launch objects with detailed information

**2. New Token List**
```
GET https://launch-mint-v1.raydium.io/get/list?sort=new&size=3&includeNsfw=false&platformId=PlatformWhiteList
```
- **Purpose**: Fetch recently created tokens
- **Query Parameters**:
  - `sort`: `new` (newest first)
  - `size`: Number of tokens (typically 3)
  - `platformId`: Filter by platform
- **Timing**: Initial page load, updated periodically
- **Response**: Array of newly created tokens

**3. Featured/Featured Tokens (Random)**
```
GET https://launch-mint-v1.raydium.io/get/random/index-top-mint
GET https://launch-mint-v1.raydium.io/get/random/index-left-mint
```
- **Purpose**: Fetch randomly featured tokens for homepage display
- **Timing**: Initial page load, updated periodically
- **Response**: Single token object for featured display

**4. Platform List**
```
GET https://launch-mint-v1.raydium.io/main/platforms-v2
```
- **Purpose**: Fetch list of supported launch platforms
- **Timing**: Initial page load
- **Response**: Array of platform information (Raydium, cointhispost, America.Fun, cook.meme, etc.)

**5. Shared Configuration (Main API)**
```
GET https://api-v3.raydium.io/mint/list
GET https://api-v3.raydium.io/main/version
GET https://api-v3.raydium.io/main/auto-fee
GET https://api-v3.raydium.io/main/rpcs
GET https://api-v3.raydium.io/mint/price?mints=...
GET https://api-v3.raydium.io/main/chain-time
```
- **Purpose**: Shared configuration and token metadata (same as liquidity pools)
- **Timing**: Initial page load
- **Response**: Standard Raydium API responses

#### POST Requests (Solana RPC)

**1. Solana RPC Calls**
```
POST https://raydium-frontend.rpcpool.com/
```
- **Purpose**: Solana blockchain RPC calls for token creation, trading, etc.
- **Method**: POST (JSON-RPC 2.0)
- **Content-Type**: `application/json`
- **Authentication**: Required (403 if not authenticated)

### 2. Periodic Updates

**Polling Pattern**:
- Token lists: Updated periodically (every 30-60 seconds)
- Featured tokens: Refreshed periodically
- New tokens: Updated frequently
- Prices: Updated via main API price endpoint

## API Response Structures

### Token Launch List Response (`/get/list`)

**Response Structure**:
```json
{
  "id": "request-id",
  "success": true,
  "data": {
    "rows": [
      {
        "mint": "token-mint-address",
        "poolId": "liquidity-pool-id",
        "configId": "curve-config-id",
        "creator": "creator-wallet-address",
        "createAt": 1762165116000,
        "name": "Token Name",
        "symbol": "TOKEN",
        "description": "Token description",
        "twitter": "https://x.com/token",
        "telegram": "https://t.me/token",
        "website": "https://token.com",
        "imgUrl": "https://ipfs.io/ipfs/...",
        "metadataUrl": "https://ipfs.io/ipfs/...",
        "platformInfo": {
          "pubKey": "platform-public-key",
          "platformClaimFeeWallet": "fee-wallet-address",
          "platformLockNftWallet": "nft-wallet-address",
          "transferFeeExtensionAuth": "auth-address",
          "cpConfigId": "config-id",
          "platformScale": "400000",
          "creatorScale": "500000",
          "burnScale": "100000",
          "feeRate": "7500",
          "creatorFeeRate": "2000",
          "name": "Platform Name",
          "web": "https://platform.com",
          "img": "https://platform-logo.png",
          "platformCurve": []
        },
        "configInfo": {
          "name": "Constant Product Curve",
          "pubKey": "config-public-key",
          "epoch": 772,
          "curveType": 0,
          "index": 0,
          "migrateFee": "0",
          "tradeFeeRate": "2500",
          "maxShareFeeRate": "10000",
          "minSupplyA": "10000000",
          "maxLockRate": "300000",
          "minSellRateA": "200000",
          "minMigrateRateA": "200000",
          "minFundRaisingB": "30000000000",
          "protocolFeeOwner": "protocol-fee-owner",
          "migrateFeeOwner": "migrate-fee-owner",
          "migrateToAmmWallet": "amm-wallet",
          "migrateToCpmmWallet": "cpmm-wallet",
          "mintB": "base-token-mint-address"
        },
        "mintB": {
          "chainId": 101,
          "address": "So11111111111111111111111111111111111111112",
          "programId": "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",
          "logoURI": "https://img-v1.raydium.io/icon/...",
          "symbol": "WSOL",
          "name": "Wrapped SOL",
          "decimals": 9,
          "tags": [],
          "extensions": {}
        },
        "decimals": 6,
        "supply": 99999999.999999,
        "marketCap": 20978.88617238405,
        "volumeA": 441615417.76112586,
        "volumeB": 452.8658825569999,
        "volumeU": 78953.77022008797,
        "finishingRate": 42.75,
        "priceStageTime1": 1762166172.365,
        "priceStageTime2": 1762170789.716,
        "priceFinalTime": 1761941432.023,
        "initPrice": "1.6875000000000105469e-7",
        "endPrice": "0.0000027",
        "totalLockedAmount": 0,
        "cliffPeriod": "0",
        "unlockPeriod": "0",
        "startTime": 0,
        "totalAllocatedShare": 0,
        "defaultCurve": false,
        "totalSellA": "80000000000000",
        "totalFundRaisingB": "54000000000",
        "migrateType": "cpmm",
        "migrateAmmId": "amm-pool-id",
        "migrateCreatorNftMint": "",
        "migratePlatformNftMint": "nft-mint-address",
        "mintProgramA": "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",
        "cpmmCreatorFeeOn": 0
      }
    ]
  }
}
```

**Key Fields**:
- `mint`: Token mint address
- `poolId`: Associated liquidity pool ID
- `creator`: Creator wallet address
- `createAt`: Creation timestamp (milliseconds)
- `name` / `symbol`: Token name and symbol
- `description`: Token description
- `twitter` / `telegram` / `website`: Social links
- `imgUrl` / `metadataUrl`: Token image and metadata (IPFS)
- `platformInfo`: Launch platform information
- `configInfo`: Bonding curve configuration
- `mintB`: Base token (usually SOL/WSOL)
- `marketCap`: Current market capitalization
- `volumeA` / `volumeB` / `volumeU`: Trading volumes
- `finishingRate`: Completion percentage of funding
- `priceStageTime1` / `priceStageTime2`: Price stage timestamps
- `initPrice` / `endPrice`: Initial and final prices
- `migrateType`: Migration type (`cpmm`, `amm`, etc.)
- `migrateAmmId`: AMM pool ID if migrated

### Platform List Response (`/main/platforms-v2`)

**Response Structure**:
```json
{
  "id": "request-id",
  "success": true,
  "data": {
    "data": [
      {
        "name": "Raydium",
        "web": "https://raydium.io/",
        "img": "https://img-v1.raydium.io/share/launch_icon.png",
        "items": [
          {
            "pubKey": "platform-public-key",
            "platformClaimFeeWallet": "fee-wallet",
            "platformLockNftWallet": "nft-wallet",
            "transferFeeExtensionAuth": "auth-address",
            "cpConfigId": "config-id",
            "platformScale": "0",
            "creatorScale": "100000",
            "burnScale": "900000",
            "feeRate": "7500",
            "creatorFeeRate": "500",
            "name": "Raydium",
            "web": "https://raydium.io/",
            "img": "https://img-v1.raydium.io/share/launch_icon.png",
            "platformCurve": []
          }
        ]
      },
      {
        "name": "@cointhispost",
        "web": "https://www.cointhispost.xyz/",
        "img": "https://ipfs.io/ipfs/...",
        "items": [ /* platform config */ ]
      }
      // ... more platforms
    ]
  }
}
```

**Key Fields**:
- `name`: Platform name
- `web`: Platform website
- `img`: Platform logo
- `items`: Array of platform configurations with fee structures

### Featured Token Response (`/get/random/index-top-mint`)

**Response Structure**:
```json
{
  "id": "request-id",
  "success": true,
  "data": {
    "data": {
      // Same structure as token launch list item
      "mint": "...",
      "poolId": "...",
      "name": "...",
      // ... full token object
    }
  }
}
```

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
4. Launchpad-Specific Data:
   - GET /main/platforms-v2 (platform list)
   - GET /get/list?sort=hotToken&size=100 (hot tokens)
   - GET /get/list?sort=new&size=3 (new tokens)
   - GET /get/random/index-top-mint (featured token)
   - GET /get/random/index-left-mint (left featured token)
   ↓
5. Asset Loading:
   - Token images (IPFS, CDN)
   - Platform logos
   - Token icons
   ↓
6. Periodic Updates:
   - Token lists (every 30-60 seconds)
   - Prices (via main API)
   - Featured tokens (periodic refresh)
```

### Real-Time Updates

**Polling Pattern**:
- Hot token list: Updated every 30-60 seconds
- New token list: Updated every 30-60 seconds
- Featured tokens: Refreshed periodically
- Prices: Updated via main API price endpoint

## POST Request Patterns

### Token Creation Flow

```
1. User Clicks "Launch Token" Button
   ↓
2. Frontend Navigates to /launchpad/create/
   ↓
3. User Fills Token Information:
   - Token name, symbol, description
   - Initial supply
   - Social links
   - Image/metadata upload
   ↓
4. Frontend Prepares Token Creation Transaction
   - Validates input
   - Uploads metadata to IPFS
   - Builds token creation instruction
   - Builds pool creation instruction
   ↓
5. POST /raydium-frontend.rpcpool.com/
   - Method: simulateTransaction
   - Simulates token creation transaction
   ↓
6. User Approves Transaction
   - Wallet prompts for signature
   ↓
7. POST /raydium-frontend.rpcpool.com/
   - Method: sendTransaction
   - Submits signed transaction to blockchain
   ↓
8. Transaction Confirmation
   - Polls RPC for transaction status
   - Updates UI on confirmation
   ↓
9. Refresh Token List
   - GET /get/list?sort=new (new token appears)
```

### Token Trading Flow

```
1. User Clicks on Token
   ↓
2. Navigation to Token Detail Page
   - Route: /launchpad/token/?mint={mintAddress}
   ↓
3. Fetch Token Details
   - GET /get/list?sort=hotToken&mint={mintAddress}
   - Or use cached data
   ↓
4. User Enters Buy/Sell Amount
   ↓
5. Frontend Prepares Trade Transaction
   - Validates amount
   - Calculates price based on bonding curve
   - Builds swap instruction
   ↓
6. POST /raydium-frontend.rpcpool.com/
   - Method: simulateTransaction
   - Simulates trade transaction
   ↓
7. User Approves Transaction
   - Wallet prompts for signature
   ↓
8. POST /raydium-frontend.rpcpool.com/
   - Method: sendTransaction
   - Submits signed transaction
   ↓
9. Transaction Confirmation
   - Polls RPC for transaction status
   - Updates token stats (volume, price)
   ↓
10. Refresh Token Data
    - GET /get/list (updates volume, price)
```

## Network Request Patterns

### Request Headers

**GET Requests**:
```
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Cache-Control: no-cache
Pragma: no-cache
Referer: https://raydium.io/launchpad/
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

## Query Parameters Reference

### Token List Endpoint (`/get/list`)

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `sort` | string | Sort order | `hotToken`, `new` |
| `size` | number | Number of tokens | `100`, `3` |
| `mintType` | string | Token type filter | `default` |
| `includeNsfw` | boolean | Include NSFW content | `false` |
| `platformId` | string | Filter by platform | `PlatformWhiteList` |

### Common Sort Options

- `hotToken`: Sort by popularity/heat
- `new`: Sort by creation time (newest first)

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
- `/get/list`
- `/get/random/index-top-mint`
- `/get/random/index-left-mint`
- `/main/platforms-v2`

**RPC Endpoint** (Authentication Required):
- `/raydium-frontend.rpcpool.com/` - Returns 403 if not authenticated
- Required for token creation and trading transactions

## Caching Strategy

**Cached Resources**:
- Token images: Cached by browser/CDN
- Platform logos: Cached with long TTL
- Static assets: Cached with long TTL

**Non-Cached Resources**:
- Token lists: Fetched fresh (real-time data)
- Featured tokens: Updated periodically
- Prices: Updated frequently

## Rate Limiting

**Observation**:
- Multiple parallel requests on page load
- Periodic polling for token lists
- Frequent updates for featured tokens
- No apparent rate limiting on public endpoints
- RPC endpoint may have rate limits

## Request Timing

**Initial Load**:
- Platform list: ~142ms
- Hot token list: ~321ms
- New token list: ~206ms
- Featured tokens: ~155-227ms

**Periodic Updates**:
- Token lists: Updated every 30-60 seconds (estimated)
- Featured tokens: Refreshed periodically

## Example: Complete Data Fetching Sequence

```
Page Load (t=0ms)
├── GET /mint/list (190ms)
├── GET /main/version (160ms)
├── GET /main/auto-fee (176ms)
└── GET /main/rpcs (159ms)

After Initial Config (t=200ms)
├── GET /main/platforms-v2 (142ms)
├── GET /get/list?sort=hotToken&size=100 (321ms)
├── GET /get/list?sort=new&size=3 (206ms)
├── GET /get/random/index-top-mint (155ms)
└── GET /get/random/index-left-mint (227ms)

Asset Loading (t=500ms)
├── Platform logos (IPFS, CDN)
├── Token images (IPFS, CDN)
└── Token icons (lazy loaded)

Periodic Updates (every 30-60 seconds)
├── GET /get/list?sort=hotToken&size=100 (token list)
├── GET /get/list?sort=new&size=3 (new tokens)
├── GET /get/random/index-top-mint (featured)
└── GET /get/random/index-left-mint (featured)
```

## Key Differences from Liquidity Pools Page

### API Structure

| Aspect | Launchpad | Liquidity Pools |
|--------|-----------|-----------------|
| **API Base** | `launch-mint-v1.raydium.io` | `api-v3.raydium.io` |
| **Main Endpoint** | `/get/list` | `/pools/info/list-v2` |
| **Data Type** | Token launches | Liquidity pools |
| **Sort Options** | `hotToken`, `new` | `desc`, `asc` |
| **Featured Items** | Random featured tokens | N/A |

### Data Structure

**Launchpad**:
- Token launch data with creator info
- Platform information
- Bonding curve configuration
- Migration status
- Funding progress

**Liquidity Pools**:
- Pool data with TVL, volume, APR
- Fee structures
- Reward information
- Pool type (Concentrated, Standard)

## Summary

Raydium's launchpad page uses:

1. **Dedicated Launchpad API** (`launch-mint-v1.raydium.io`) for:
   - Token launch list (hot tokens, new tokens)
   - Platform information
   - Featured/random tokens
   - Token creation and trading data

2. **Main Raydium API** (`api-v3.raydium.io`) for:
   - Shared configuration
   - Token metadata
   - Price information
   - RPC endpoints

3. **Solana RPC** (`raydium-frontend.rpcpool.com`) for:
   - Token creation transactions
   - Trading transactions
   - Account queries
   - Transaction simulation

4. **Data Flow**:
   - **GET requests**: Fetch token lists, platform info, featured tokens
   - **POST requests**: Solana RPC calls for token creation and trading
   - **Polling**: Periodic updates for token lists and featured items
   - **Lazy loading**: Images and assets loaded on demand

The page primarily **fetches data** (GET) and uses **POST requests** only for Solana RPC calls to interact with the blockchain for token creation and trading operations. All token launch data is retrieved via REST API endpoints from the dedicated launchpad service.

