# WooFi Earn - Network Fetching & Posting Analysis

This document analyzes how [WooFi's earn page](https://woofi.com/swap/earn) fetches and posts data, including all API endpoints, request/response structures, and data flow patterns.

## Overview

WooFi's earn page displays yield farming vaults across multiple networks, allowing users to deposit assets and earn passive yield. The page supports multiple blockchain networks and displays featured vaults, yield statistics, and token information.

## Key API Endpoints

### Base API URLs
- **WooFi API**: `https://api.woofi.com/`
- **Binance API**: `https://api.binance.com/` (price data)
- **WooFi CDN**: `https://oss.woo.org/` (images and assets)
- **Third-Party**: CoinGecko (via WooFi proxy), 1inch (via WooFi proxy)

## Network Requests Breakdown

### 1. Initial Page Load Requests

#### GET Requests (Data Fetching)

**1. Featured Vaults**
```
GET https://api.woofi.com/feature_vaults
```
- **Purpose**: Fetch featured/highlighted vaults for homepage display
- **Timing**: Initial page load
- **Method**: XMLHttpRequest
- **Response**: Array of featured vault objects with network, symbol, vault address, and APR

**2. Yield Data by Network**
```
GET https://api.woofi.com/yield?&network={network}
```
- **Purpose**: Fetch yield farming data for a specific network
- **Query Parameters**:
  - `network`: Network identifier (arbitrum, base, avax, mantle, zksync, optimism, polygon, bsc, linea, sonic, berachain)
- **Timing**: Initial page load (called for each supported network in parallel)
- **Method**: XMLHttpRequest
- **Response**: Yield data including auto-compounding vaults, APY, TVL, and detailed metrics

**3. Token Lists by Network (1inch Integration)**
```
GET https://api.woofi.com/1inch_tokens?network={network}
```
- **Purpose**: Fetch token lists for each network (powered by 1inch)
- **Query Parameters**:
  - `network`: Network identifier (ethereum, arbitrum, base, avax, zksync, optimism, polygon, bsc)
- **Timing**: Initial page load (called for each supported network in parallel)
- **Method**: XMLHttpRequest
- **Response**: Token metadata including address, symbol, decimals, name, logoURI, and tags

**4. Coin Market Data (CoinGecko Integration)**
```
GET https://api.woofi.com/coingecko/coins_markets?page={page}
```
- **Purpose**: Fetch cryptocurrency market data from CoinGecko
- **Query Parameters**:
  - `page`: Page number for pagination (1, 2, 3, etc.)
- **Timing**: Initial page load (multiple pages fetched in parallel)
- **Method**: XMLHttpRequest
- **Response**: Market data including prices, market cap, volume, and price changes

**5. OKX Tokens**
```
GET https://api.woofi.com/okx_tokens
```
- **Purpose**: Fetch token list from OKX exchange
- **Timing**: Initial page load
- **Method**: XMLHttpRequest
- **Response**: Token list data

**6. Price Data**
```
GET https://api.binance.com/api/v3/ticker/price?symbol=WOOUSDT
```
- **Purpose**: Fetch WOO token price from Binance
- **Query Parameters**:
  - `symbol`: Trading pair symbol (WOOUSDT)
- **Timing**: Initial page load, updated periodically
- **Method**: Fetch
- **Response**: Price data for the specified trading pair

**7. Claim Data (User-Specific)**
```
GET https://api.woofi.com/avalanche_boost_plan/claim?user_address={address}
```
- **Purpose**: Fetch claimable rewards for a specific user address
- **Query Parameters**:
  - `user_address`: User's wallet address
- **Timing**: When wallet is connected
- **Method**: XMLHttpRequest
- **Response**: Claimable reward data

#### POST Requests

**1. Analytics Tracking**
```
POST https://api2.amplitude.com/2/httpapi
```
- **Purpose**: User analytics and event tracking
- **Timing**: On user interactions and page events
- **Method**: POST
- **Content-Type**: `application/json`

**2. Google Analytics**
```
POST https://www.google-analytics.com/g/collect
```
- **Purpose**: Google Analytics tracking
- **Timing**: On page load and user interactions
- **Method**: POST (via fetch)

### 2. Asset Loading

**Images and Icons**:
- Vault icons: `https://oss.woo.org/static/images/sdk/`
- Network logos: `https://oss.woo.org/static/images/sdk/{network}.png`
- Token logos: Via 1inch token list (logoURI field)
- Custom images: `https://woofi.com/swap/`

## API Response Structures

### Featured Vaults Response (`/feature_vaults`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": [
    {
      "network": "arbitrum",
      "network_display_name": "Arbitrum",
      "symbol": "ETH",
      "vault_address": "0xba452bCc4BC52AF2fe1190e7e1dBE267ad1C2d08",
      "apr": 5.514177962795687
    },
    {
      "network": "avax",
      "network_display_name": "Avalanche",
      "symbol": "USDC",
      "vault_address": "0x11B29AE3037F4526e4AA56952318e0d01ADA836A",
      "apr": 8.4228697197254
    }
    // ... more featured vaults
  ]
}
```

**Key Fields**:
- `network`: Network identifier
- `network_display_name`: Human-readable network name
- `symbol`: Token symbol
- `vault_address`: Smart contract address of the vault
- `apr`: Annual Percentage Rate

### Yield Data Response (`/yield?&network={network}`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": {
    "auto_compounding": {
      "0xba452bCc4BC52AF2fe1190e7e1dBE267ad1C2d08": {
        "symbol": "ETH",
        "source": "woofi_super_charger",
        "apy": 5.668602032696568,
        "tvl": "884626632167290098941952",
        "price": 3857.6075822816,
        "share_price": "1096065192526723773",
        "decimals": 18,
        "loan_assets_percentage": 84.853563964617,
        "loan_interest_apr": 6.140000000000001,
        "reserve_vault_assets_percentage": 15.146436035383006,
        "reserve_vault_apr": 2.0081894820494797,
        "weighted_average_apr": 5.514177962795397,
        "x_woo_rewards_apr": 0,
        "woo_rewards_apr": 0,
        "arb_rewards_apr": 2.899660924082734e-13,
        "reward_apr": 2.899660924082734e-13,
        "liquid_staking_apr": 0
      }
      // ... more vaults
    },
    "total_deposit": "2049701420656945830821888"
  }
}
```

**Key Fields**:
- `auto_compounding`: Object mapping vault addresses to vault data
  - `symbol`: Token symbol
  - `source`: Vault source/provider
  - `apy`: Annual Percentage Yield
  - `tvl`: Total Value Locked (in token units)
  - `price`: Token price in USD
  - `share_price`: Vault share price
  - `decimals`: Token decimals
  - `loan_assets_percentage`: Percentage of assets in loans
  - `loan_interest_apr`: APR from loan interest
  - `reserve_vault_assets_percentage`: Percentage in reserve vault
  - `reserve_vault_apr`: APR from reserve vault
  - `weighted_average_apr`: Overall weighted APR
  - `x_woo_rewards_apr`: xWOO rewards APR
  - `woo_rewards_apr`: WOO rewards APR
  - `arb_rewards_apr`: ARB rewards APR
  - `reward_apr`: Total reward APR
  - `liquid_staking_apr`: Liquid staking APR
- `total_deposit`: Total deposits across all vaults

### Token List Response (`/1inch_tokens?network={network}`)

**Response Structure**:
```json
{
  "tokens": {
    "0x32eb7902d4134bf98a28b963d26de779af92a212": {
      "address": "0x32eb7902d4134bf98a28b963d26de779af92a212",
      "symbol": "rDPX",
      "decimals": 18,
      "name": "Dopex Rebate Token",
      "logoURI": "https://tokens.1inch.io/0x32eb7902d4134bf98a28b963d26de779af92a212.png",
      "eip2612": false,
      "tags": [
        "crosschain",
        "RISK:norisk",
        "RISK:unverified",
        "tokens"
      ]
    }
    // ... more tokens
  }
}
```

**Key Fields**:
- `address`: Token contract address
- `symbol`: Token symbol
- `decimals`: Token decimals
- `name`: Token name
- `logoURI`: Logo image URL
- `eip2612`: Whether token supports EIP-2612 (permit function)
- `tags`: Array of token tags (risk levels, categories, etc.)

### Coin Market Data Response (`/coingecko/coins_markets?page={page}`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": [
    {
      "id": "bitcoin",
      "symbol": "btc",
      "name": "Bitcoin",
      "image": "https://coin-images.coingecko.com/coins/images/1/large/bitcoin.png",
      "current_price": 107058,
      "market_cap": 2135522920340,
      "market_cap_rank": 1,
      "fully_diluted_valuation": 2135522920340,
      "total_volume": 74614459407,
      "high_24h": 108151,
      "low_24h": 105540,
      "price_change_24h": -1081.27488562277,
      "price_change_percentage_24h": -0.99989,
      "market_cap_change_24h": -21083503884.953857,
      "market_cap_change_percentage_24h": -0.97762,
      "circulating_supply": 19944128.0,
      "total_supply": 19944128.0,
      "max_supply": 21000000.0,
      "ath": 126080,
      "ath_change_percentage": -15.06574,
      "ath_date": "2025-10-06T18:57:42.558Z",
      "atl": 67.81,
      "atl_change_percentage": 157821.54914,
      "atl_date": "2013-07-06T00:00:00.000Z",
      "roi": null,
      "last_updated": "2025-11-04T03:52:12.771Z"
    }
    // ... more coins
  ]
}
```

**Key Fields**:
- `id`: CoinGecko coin ID
- `symbol`: Token symbol
- `name`: Token name
- `image`: Coin image URL
- `current_price`: Current price in USD
- `market_cap`: Market capitalization
- `market_cap_rank`: Market cap ranking
- `total_volume`: 24h trading volume
- `high_24h` / `low_24h`: 24h price range
- `price_change_24h`: 24h price change
- `price_change_percentage_24h`: 24h price change percentage
- `circulating_supply`: Circulating supply
- `ath` / `ath_date`: All-time high price and date
- `atl` / `atl_date`: All-time low price and date

### Price Data Response (`/api/v3/ticker/price`)

**Response Structure**:
```json
{
  "symbol": "WOOUSDT",
  "price": "0.03000000"
}
```

**Key Fields**:
- `symbol`: Trading pair symbol
- `price`: Current price

## Data Fetching Flow

### Initial Page Load Sequence

```
1. Page HTML Loads
   ↓
2. JavaScript Bundles Load
   ↓
3. Initial API Calls (Parallel):
   - GET /feature_vaults (featured vaults)
   - GET /yield?&network=arbitrum
   - GET /yield?&network=base
   - GET /yield?&network=avax
   - GET /yield?&network=mantle
   - GET /yield?&network=zksync
   - GET /yield?&network=optimism
   - GET /yield?&network=polygon
   - GET /yield?&network=bsc
   - GET /yield?&network=linea
   - GET /yield?&network=sonic
   - GET /yield?&network=berachain
   ↓
4. Token Lists (Parallel):
   - GET /1inch_tokens?network=ethereum
   - GET /1inch_tokens?network=arbitrum
   - GET /1inch_tokens?network=base
   - GET /1inch_tokens?network=avax
   - GET /1inch_tokens?network=zksync
   - GET /1inch_tokens?network=optimism
   - GET /1inch_tokens?network=polygon
   - GET /1inch_tokens?network=bsc
   ↓
5. Market Data (Parallel):
   - GET /coingecko/coins_markets?page=1
   - GET /coingecko/coins_markets?page=2
   - GET /coingecko/coins_markets?page=3
   ↓
6. Additional Data:
   - GET /okx_tokens
   - GET /api.binance.com/api/v3/ticker/price?symbol=WOOUSDT
   ↓
7. Asset Loading:
   - Network logos
   - Token icons
   - Vault images
```

### Real-Time Updates

**Polling Pattern**:
- Yield data: Updated periodically (every 30-60 seconds)
- Price data: Updated frequently (every 5-10 seconds)
- Featured vaults: Updated periodically

## Supported Networks

**EVM Chains**:
- Arbitrum
- Base
- Avalanche (AVAX)
- Mantle
- zkSync
- Optimism
- Polygon
- BNB Chain (BSC)
- Linea
- Sonic
- Berachain
- Ethereum (mainnet)

## POST Request Patterns

### Deposit/Withdraw Flow (Inferred)

When users interact with vaults, the following flow likely occurs:

```
1. User Clicks on Vault
   ↓
2. Frontend Displays Vault Details
   - Uses cached yield data
   - Shows APY, TVL, and other metrics
   ↓
3. User Connects Wallet
   ↓
4. Frontend Fetches User Position
   - May use blockchain RPC calls
   - Gets user's deposit balance
   - Gets user's claimable rewards
   ↓
5. User Enters Deposit Amount
   ↓
6. Frontend Prepares Transaction
   - Validates amount
   - Calculates expected shares
   - Builds transaction
   ↓
7. User Approves Token (if needed)
   - ERC-20 approve transaction
   ↓
8. User Approves Deposit Transaction
   - Wallet prompts for signature
   ↓
9. Transaction Submission
   - Smart contract interaction
   - Deposits tokens into vault
   ↓
10. Transaction Confirmation
    - Polls blockchain for confirmation
    - Updates UI
    ↓
11. Refresh Data
    - GET /yield?&network={network} (updates TVL)
    - Fetch user position (updates balance)
```

### Claim Rewards Flow (Inferred)

```
1. User Clicks "Claim Rewards"
   ↓
2. Frontend Checks Claimable Rewards
   - GET /avalanche_boost_plan/claim?user_address={address}
   - Or checks on-chain data
   ↓
3. User Approves Claim Transaction
   - Wallet prompts for signature
   ↓
4. Transaction Submission
   - Smart contract interaction
   - Claims rewards
   ↓
5. Transaction Confirmation
   - Polls blockchain for confirmation
   - Updates UI
```

## Network Request Patterns

### Request Headers

**GET Requests**:
```
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Cache-Control: no-cache
Pragma: no-cache
Referer: https://woofi.com/swap/earn
```

**POST Requests**:
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

### Yield Endpoint (`/yield`)

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `network` | string | Network identifier | `arbitrum`, `base`, `avax` |

### Supported Networks

- `arbitrum`: Arbitrum One
- `base`: Base
- `avax`: Avalanche
- `mantle`: Mantle
- `zksync`: zkSync Era
- `optimism`: Optimism
- `polygon`: Polygon
- `bsc`: BNB Chain
- `linea`: Linea
- `sonic`: Sonic
- `berachain`: Berachain

### CoinGecko Endpoint (`/coingecko/coins_markets`)

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `page` | number | Page number for pagination | `1`, `2`, `3` |

### 1inch Tokens Endpoint (`/1inch_tokens`)

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `network` | string | Network identifier | `ethereum`, `arbitrum`, `base` |

### Claim Endpoint (`/avalanche_boost_plan/claim`)

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `user_address` | string | User's wallet address | `0x...` |

## Error Handling

**API Error Responses**:
```json
{
  "status": "error",
  "message": "Error message",
  "code": "ERROR_CODE"
}
```

**Success Response**:
```json
{
  "status": "ok",
  "data": { /* response data */ }
}
```

## Authentication

**Public Endpoints** (No Authentication):
- `/feature_vaults`
- `/yield`
- `/1inch_tokens`
- `/coingecko/coins_markets`
- `/okx_tokens`

**User-Specific Endpoints**:
- `/avalanche_boost_plan/claim?user_address={address}` - Requires wallet address (no API key needed)

**Third-Party APIs**:
- Binance API: Public, no authentication required
- CoinGecko (via proxy): Public, no authentication required
- 1inch (via proxy): Public, no authentication required

## Caching Strategy

**Cached Resources**:
- Token metadata: Cached by browser/CDN
- Network logos: Cached with long TTL
- Static assets: Cached with long TTL

**Non-Cached Resources**:
- Yield data: Fetched fresh (real-time data)
- Prices: Updated frequently
- Featured vaults: Updated periodically

## Rate Limiting

**Observation**:
- Multiple parallel requests on page load (10+ networks)
- Periodic polling for yield data
- Frequent price updates
- No apparent rate limiting on public endpoints
- Third-party APIs (Binance, CoinGecko) may have rate limits

## Request Timing

**Initial Load**:
- Featured vaults: ~123ms
- Yield data per network: ~121-1009ms (varies by network)
- Token lists: ~409-1299ms (varies by network)
- Market data: ~380-707ms per page
- WOO price: ~165ms

**Total Initial Load**:
- Approximately 50-100 API requests in parallel
- Total load time: ~1-3 seconds (depending on network speed)

## Example: Complete Data Fetching Sequence

```
Page Load (t=0ms)
├── GET /feature_vaults (123ms)
├── GET /yield?&network=arbitrum (196ms)
├── GET /yield?&network=base (265ms)
├── GET /yield?&network=avax (121ms)
├── GET /yield?&network=mantle (259ms)
├── GET /yield?&network=zksync (1009ms)
├── GET /yield?&network=optimism (197ms)
├── GET /yield?&network=polygon (200ms)
├── GET /yield?&network=bsc (253ms)
├── GET /yield?&network=linea (200ms)
├── GET /yield?&network=sonic (254ms)
├── GET /yield?&network=berachain (262ms)
├── GET /1inch_tokens?network=ethereum (1299ms)
├── GET /1inch_tokens?network=arbitrum (754ms)
├── GET /1inch_tokens?network=base (573ms)
├── GET /1inch_tokens?network=avax (553ms)
├── GET /1inch_tokens?network=zksync (563ms)
├── GET /1inch_tokens?network=optimism (570ms)
├── GET /1inch_tokens?network=polygon (550ms)
├── GET /1inch_tokens?network=bsc (834ms)
├── GET /coingecko/coins_markets?page=1 (636ms)
├── GET /coingecko/coins_markets?page=2 (380ms)
├── GET /coingecko/coins_markets?page=3 (707ms)
├── GET /okx_tokens (409ms)
└── GET /api.binance.com/api/v3/ticker/price?symbol=WOOUSDT (165ms)

After Initial Load (t=2000ms+)
├── Periodic yield data updates
├── Price updates
└── User-specific data (when wallet connected)
```

## Key Features

### Multi-Network Support

WooFi Earn supports **11+ blockchain networks**, fetching yield data for each network in parallel.

### Comprehensive Token Data

- Token lists from 1inch (8+ networks)
- Market data from CoinGecko (pagination)
- Exchange tokens from OKX
- Price data from Binance

### Yield Aggregation

- Auto-compounding vaults
- Multiple yield sources (loan interest, reserve vault, rewards)
- Weighted average APR calculations
- Real-time TVL tracking

### User-Specific Features

- Claimable rewards tracking
- User position data (via blockchain)
- Network-specific boost plans

## Summary

WooFi's earn page uses:

1. **WooFi API** (`api.woofi.com`) for:
   - Featured vaults
   - Yield data per network
   - Token lists (1inch integration)
   - Market data (CoinGecko integration)
   - User claim data

2. **Third-Party APIs** for:
   - Price data (Binance)
   - Token metadata (1inch via proxy)
   - Market data (CoinGecko via proxy)
   - Exchange tokens (OKX via proxy)

3. **Data Flow**:
   - **GET requests**: Fetch vault data, yield data, token lists, market data, prices
   - **POST requests**: Analytics tracking, blockchain transactions (wallet interactions)
   - **Multi-network**: Parallel fetching for 11+ networks
   - **Real-time updates**: Periodic polling for yield and price data

The page primarily **fetches data** (GET) for display purposes. POST requests are mainly used for analytics tracking. Actual vault interactions (deposit/withdraw/claim) are handled through blockchain transactions via wallet connections, which don't appear in the HTTP network requests but are handled by Web3 providers.

