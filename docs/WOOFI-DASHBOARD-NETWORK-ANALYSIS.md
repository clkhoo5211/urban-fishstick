# WooFi Dashboard - Network Fetching & Posting Analysis

This document analyzes how [WooFi's dashboard page](https://woofi.com/swap/dashboard) fetches and posts data, including all API endpoints, request/response structures, and data flow patterns.

## Overview

WooFi's dashboard page displays comprehensive analytics and statistics about the platform, including trading volume, number of trades, traders, turnover rates, source aggregators, token statistics, and cumulative metrics. The page provides real-time data visualization across multiple networks.

## Key API Endpoints

### Base API URLs
- **WooFi API**: `https://api.woofi.com/`
- **Binance API**: `https://api.binance.com/` (price data)
- **WooFi CDN**: `https://oss.woo.org/` (images and assets)
- **Third-Party**: CoinGecko (via WooFi proxy), 1inch (via WooFi proxy)

## Network Requests Breakdown

### 1. Initial Page Load Requests

#### GET Requests (Data Fetching)

**1. Statistics by Period (`/stat`)**
```
GET https://api.woofi.com/stat?period={period}&network={network}
```
- **Purpose**: Fetch trading statistics over a specific time period
- **Timing**: Initial page load, updated periodically
- **Method**: XMLHttpRequest
- **Parameters**:
  - `period`: Time period (e.g., `1m` for 1 month)
  - `network`: Network identifier (e.g., `arbitrum`)
- **Response**: Array of daily statistics including volume, traders, transactions, and turnover rate

**2. Source Statistics (`/source_stat`)**
```
GET https://api.woofi.com/source_stat?period={period}&network={network}
```
- **Purpose**: Fetch trading volume breakdown by source aggregator
- **Timing**: Initial page load, updated periodically
- **Method**: XMLHttpRequest
- **Parameters**:
  - `period`: Time period (e.g., `1m` for 1 month)
  - `network`: Network identifier (e.g., `arbitrum`)
- **Response**: Array of source aggregator statistics (0x, 1inch, KyberSwap, ODOS, Velora, OpenOcean, etc.)

**3. Token Statistics (`/token_stat`)**
```
GET https://api.woofi.com/token_stat?network={network}
```
- **Purpose**: Fetch token-level statistics (volume, trades, liquidity, turnover rate)
- **Timing**: Initial page load, updated periodically
- **Method**: XMLHttpRequest
- **Parameters**:
  - `network`: Network identifier (e.g., `arbitrum`)
- **Response**: Object with network-keyed token statistics arrays

**4. Cumulative Statistics (`/cumulate_stat`)**
```
GET https://api.woofi.com/cumulate_stat?period={period}&network={network}
```
- **Purpose**: Fetch cumulative statistics over a time period
- **Timing**: Initial page load, updated periodically
- **Method**: XMLHttpRequest
- **Parameters**:
  - `period`: Time period (e.g., `1m` for 1 month)
  - `network`: Network identifier (e.g., `arbitrum`)
- **Response**: Cumulative statistics including 24h volume, traders, transactions, and turnover rate

**5. Total Statistics (`/total_stat`)**
```
GET https://api.woofi.com/total_stat?network={network}
```
- **Purpose**: Fetch total platform statistics for a specific network
- **Timing**: Initial page load, updated periodically
- **Method**: XMLHttpRequest
- **Parameters**:
  - `network`: Network identifier (e.g., `arbitrum`)
- **Response**: Total volume, traders, transactions, and update timestamp

**6. Multi-Total Statistics (`/multi_total_stat`)**
```
GET https://api.woofi.com/multi_total_stat
```
- **Purpose**: Fetch aggregated statistics across all networks
- **Timing**: Initial page load, updated periodically
- **Method**: XMLHttpRequest
- **Response**: Cross-network totals including volume, traders, transactions, WOO buyback, cross-chain swaps

**7. Shared Configuration (Same as Other Pages)**
```
GET https://api.woofi.com/okx_tokens
GET https://api.woofi.com/1inch_tokens?network={network}
GET https://api.woofi.com/coingecko/coins_markets?page={page}
GET https://api.binance.com/api/v3/ticker/price?symbol=WOOUSDT
```
- **Purpose**: Token lists, market data, and price information
- **Timing**: Initial page load
- **Method**: XMLHttpRequest/Fetch

#### POST Requests

**1. Analytics Tracking**
```
POST https://api2.amplitude.com/2/httpapi
POST https://www.google-analytics.com/g/collect
```
- **Purpose**: User analytics and event tracking
- **Timing**: On user interactions and page events
- **Method**: POST

### 2. Asset Loading

**Images**:
- Network logos: `https://oss.woo.org/static/images/sdk/` (arbitrum, mantle, zksync_era, optimism, linea, hype, bera)
- Token icons: `https://oss.woo.org/static/icons/` (USDC, USDT, ETH, WBTC, USDC.e, ARB, etc.)

## API Response Structures

### Statistics Response (`/stat`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": [
    {
      "id": "20367",
      "timestamp": "1759708800",
      "volume_usd": "4331926747301000000000000",
      "traders": "1089",
      "txs": "2904",
      "txns": "2904",
      "turnover_rate_percentage": 363.5461506786437
    }
    // ... more daily entries
  ]
}
```

**Key Fields**:
- `id`: Unique record identifier
- `timestamp`: Unix timestamp for the day
- `volume_usd`: Trading volume in USD (raw units, requires division by 1e18)
- `traders`: Number of unique traders
- `txs` / `txns`: Number of transactions (synonyms)
- `turnover_rate_percentage`: Turnover rate as a percentage

**Calculation**:
- Volume in human-readable format: `volume_usd / 1e18`
- Example: `4331926747301000000000000 / 1e18 = $4,331,926,747.30`

### Source Statistics Response (`/source_stat`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": [
    {
      "id": "0",
      "name": "WOOFi",
      "volume_usd": "39938420564000000000000",
      "txns": 46,
      "percentage": 0.14140212821012052
    },
    {
      "id": "1",
      "name": "1inch",
      "volume_usd": "3085136015863000000000000",
      "txns": 153,
      "percentage": 10.922935666963907
    },
    {
      "id": "11",
      "name": "0x",
      "volume_usd": "14919870652464000000000000",
      "txns": 8769,
      "percentage": 52.82385815676916
    }
    // ... more sources
  ]
}
```

**Key Fields**:
- `id`: Source aggregator identifier
- `name`: Source aggregator name (WOOFi, 1inch, 0x, KyberSwap, ODOS, Velora, OpenOcean, YieldYak, Other)
- `volume_usd`: Trading volume in USD (raw units, requires division by 1e18)
- `txns`: Number of transactions
- `percentage`: Percentage of total volume

**Source Aggregator IDs**:
- `0`: WOOFi (native)
- `1`: 1inch
- `3`: OpenOcean
- `5`: YieldYak
- `8`: Velora
- `11`: 0x
- `12`: ODOS
- `23`: KyberSwap
- `99`: Other

### Token Statistics Response (`/token_stat`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": {
    "arbitrum": [
      {
        "logo_url": "https://oss.woo.org/static/icons/USDC.png",
        "symbol": "USDC",
        "decimals": 6,
        "tvl": "130954645551",
        "24h_volume_usd": "2717999735891000000000000",
        "24h_txs": "1701",
        "24h_txns": "1701",
        "turnover_rate_percentage": 2075.527541962977
      }
      // ... more tokens
    ]
    // ... more networks
  }
}
```

**Key Fields**:
- `logo_url`: Token logo URL
- `symbol`: Token symbol (USDC, USDT, ETH, WBTC, USDC.e, ARB, etc.)
- `decimals`: Token decimals (6 for USDC/USDT, 18 for ETH/ARB, 8 for WBTC)
- `tvl`: Total Value Locked (in token's native units)
- `24h_volume_usd`: 24-hour trading volume in USD (raw units, requires division by 1e18)
- `24h_txs` / `24h_txns`: 24-hour transaction count
- `turnover_rate_percentage`: 24-hour turnover rate as a percentage

**Calculation**:
- Volume in human-readable format: `24h_volume_usd / 1e18`
- TVL in human-readable format: `tvl / (10 ** decimals)`
- Example: `2717999735891000000000000 / 1e18 = $2,717,999,735.89`

### Cumulative Statistics Response (`/cumulate_stat`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": {
    "24h_volume_usd": "3961591407866000000000000",
    "24h_traders": "579",
    "24h_txs": "1478",
    "24h_txns": "1478",
    "24h_turnover_rate_percentage": 426.2338251288737
  }
}
```

**Key Fields**:
- `24h_volume_usd`: 24-hour cumulative volume in USD (raw units, requires division by 1e18)
- `24h_traders`: 24-hour unique traders
- `24h_txs` / `24h_txns`: 24-hour transaction count
- `24h_turnover_rate_percentage`: 24-hour turnover rate as a percentage

**Calculation**:
- Volume in human-readable format: `24h_volume_usd / 1e18`
- Example: `3961591407866000000000000 / 1e18 = $26,018,595.00`

### Total Statistics Response (`/total_stat`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": {
    "volume_usd": "7644960850117566000000000000",
    "traders": "777836",
    "txs": "4319671",
    "txns": "4319671",
    "updated_at": "1762228975"
  }
}
```

**Key Fields**:
- `volume_usd`: Total volume in USD (raw units, requires division by 1e18)
- `traders`: Total unique traders
- `txs` / `txns`: Total transactions
- `updated_at`: Unix timestamp of last update

**Calculation**:
- Volume in human-readable format: `volume_usd / 1e18`
- Example: `7644960850117566000000000000 / 1e18 = $7,644,960,850,117.57`

### Multi-Total Statistics Response (`/multi_total_stat`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": {
    "past_24h_volume": "26018595837865532428610504",
    "yesterday_volume_usd": "30164017456998189547714501",
    "last_week_volume_usd": "163083575432197539185016379",
    "total_volume_usd": "32510184752661823562905460067",
    "woo_buyback_volume": "39460918237173608610839077",
    "cross_chain_swap_txns": 4251788,
    "updated_at": 1762229129
  }
}
```

**Key Fields**:
- `past_24h_volume`: Past 24-hour volume (raw units, requires division by 1e18)
- `yesterday_volume_usd`: Yesterday's volume (raw units, requires division by 1e18)
- `last_week_volume_usd`: Last week's volume (raw units, requires division by 1e18)
- `total_volume_usd`: Total all-time volume (raw units, requires division by 1e18)
- `woo_buyback_volume`: Total WOO buyback volume (raw units, requires division by 1e18)
- `cross_chain_swap_txns`: Total cross-chain swap transactions
- `updated_at`: Unix timestamp of last update

**Calculation**:
- All volumes in human-readable format: `{field} / 1e18`
- Example: `26018595837865532428610504 / 1e18 = $26,018,595.84`
- Example: `32510184752661823562905460067 / 1e18 = $32,510,184,752.66`
- Example: `39460918237173608610839077 / 1e18 = 39,460,918.24 WOO`

## Data Fetching Flow

### Initial Page Load Sequence

```
1. Page HTML Loads
   ↓
2. JavaScript Bundles Load
   ↓
3. Dashboard-Specific API Calls (Parallel):
   - GET /stat?period=1m&network=arbitrum (daily statistics)
   - GET /source_stat?period=1m&network=arbitrum (source breakdown)
   - GET /token_stat?network=arbitrum (token statistics)
   - GET /cumulate_stat?period=1m&network=arbitrum (cumulative stats)
   - GET /total_stat?network=arbitrum (network totals)
   - GET /multi_total_stat (cross-network totals)
   ↓
4. Shared Configuration (Parallel):
   - GET /okx_tokens
   - GET /1inch_tokens?network={network} (multiple networks)
   - GET /coingecko/coins_markets?page={page} (multiple pages)
   - GET /api.binance.com/api/v3/ticker/price?symbol=WOOUSDT
   ↓
5. Asset Loading:
   - Network logos
   - Token icons
```

### Real-Time Updates

**Polling Pattern**:
- Statistics: Updated periodically (every 30-60 seconds)
- Source statistics: Updated periodically
- Token statistics: Updated periodically
- Cumulative statistics: Updated periodically
- Total statistics: Updated periodically
- Multi-total statistics: Updated periodically

**Network Filtering**:
- Users can filter by network (Arbitrum, Mantle, zkSync Era, Optimism, Linea, Hype, Berachain, etc.)
- Filtering triggers new API calls with updated `network` parameter

**Period Filtering**:
- Users can filter by time period (1M, 3M, 1Y, etc.)
- Period filtering triggers new API calls with updated `period` parameter

## Query Parameters Reference

### Statistics Endpoints

**Period Values**:
- `1m`: 1 month
- `3m`: 3 months
- `1y`: 1 year
- `all`: All time

**Network Values**:
- `arbitrum`: Arbitrum
- `mantle`: Mantle
- `zksync`: zkSync Era
- `optimism`: Optimism
- `linea`: Linea
- `hype`: Hype
- `bera`: Berachain
- `bsc`: BSC
- `ethereum`: Ethereum
- `base`: Base
- `avax`: Avalanche
- `polygon`: Polygon

**Example Requests**:
```
GET /stat?period=1m&network=arbitrum
GET /source_stat?period=1m&network=arbitrum
GET /token_stat?network=arbitrum
GET /cumulate_stat?period=1m&network=arbitrum
GET /total_stat?network=arbitrum
GET /multi_total_stat
```

## Data Display Features

### Overview Statistics

**Top Banner**:
- 24-hour Trading Volume: `$26,018,595`
- Total Volume: `$32,510,184,752`
- Total Bought Back: `39,460,918 WOO`

**Network Filter**:
- Displays available networks as clickable icons
- Default: Arbitrum
- Switching networks updates all statistics

### Trading Volume Chart

**Chart Data**:
- X-axis: Time (daily data points)
- Y-axis: Volume (USD)
- Period selector: 1M, 3M, 1Y, All
- Data source: `/stat?period={period}&network={network}`

**Source Breakdown**:
- Pie chart or bar chart showing volume by source aggregator
- Sources: 0x, 1inch, KyberSwap, ODOS, Velora, OpenOcean, Others
- Data source: `/source_stat?period={period}&network={network}`

### Key Metrics Cards

**Number of Addresses**:
- Past 24H: `579`
- Data source: `/cumulate_stat?period=1m&network={network}` → `24h_traders`

**Number of Trades**:
- Past 24H: `1,478`
- Data source: `/cumulate_stat?period=1m&network={network}` → `24h_txs`

**Turnover Rate**:
- Past 24H: `426.2%`
- Data source: `/cumulate_stat?period=1m&network={network}` → `24h_turnover_rate_percentage`

### Token Statistics Table

**Table Columns**:
- Assets: Token symbol with logo
- Network: Network name
- Total Volume (24h): Trading volume in USD
- Number of Trades (24H): Transaction count
- Liquidity: Total Value Locked (TVL)

**Sortable Columns**:
- Total Volume (24h): Sortable
- Number of Trades (24H): Sortable
- Liquidity: Sortable

**Data Source**:
- `/token_stat?network={network}` → Returns array of tokens with statistics

**Example Tokens**:
1. USDC (Arbitrum): $2,717,999 volume, 1,701 trades, $130,954 liquidity
2. USDT (Arbitrum): $1,863,190 volume, 1,566 trades, $67,633 liquidity
3. ETH (Arbitrum): $1,531,296 volume, 380 trades, $322,716 liquidity
4. WBTC (Arbitrum): $1,402,032 volume, 460 trades, $233,134 liquidity
5. USDC.e (Arbitrum): $244,076 volume, 170 trades, $47,025 liquidity
6. ARB (Arbitrum): $164,587 volume, 93 trades, $127,976 liquidity

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
- All dashboard statistics endpoints are public
- No authentication required
- No rate limiting apparent

**Third-Party APIs**:
- Binance API: Public, no authentication required
- CoinGecko (via proxy): Public, no authentication required
- 1inch (via proxy): Public, no authentication required

## Caching Strategy

**Cached Resources**:
- Network logos: Cached by browser/CDN
- Token icons: Cached by browser/CDN
- Static assets: Cached with long TTL

**Non-Cached Resources**:
- Statistics: Fetched fresh (real-time data)
- Source statistics: Updated periodically
- Token statistics: Updated periodically
- Cumulative statistics: Updated periodically
- Total statistics: Updated periodically

## Rate Limiting

**Observation**:
- Multiple parallel requests on page load
- Periodic polling for statistics updates
- Network/period filter changes trigger new requests
- No apparent rate limiting on public endpoints
- Third-party APIs may have rate limits

## Request Timing

**Initial Load**:
- Statistics: ~108-215ms
- Source statistics: ~222ms
- Token statistics: ~1297ms (slowest)
- Cumulative statistics: ~343ms
- Total statistics: ~118ms
- Multi-total statistics: ~491ms
- Token lists: ~283-1317ms (varies by network)
- Market data: ~212-524ms per page

**Total Initial Load**:
- Approximately 15-20 dashboard-specific API requests
- Total load time: ~1-3 seconds (depending on network speed)

## Example: Complete Data Fetching Sequence

```
Page Load (t=0ms)
├── GET /stat?period=1m&network=arbitrum (108ms)
├── GET /source_stat?period=1m&network=arbitrum (222ms)
├── GET /token_stat?network=arbitrum (1297ms)
├── GET /cumulate_stat?period=1m&network=arbitrum (343ms)
├── GET /total_stat?network=arbitrum (118ms)
├── GET /multi_total_stat (491ms)
├── GET /okx_tokens (283ms)
├── GET /1inch_tokens?network=ethereum (1317ms)
├── GET /1inch_tokens?network=arbitrum (634ms)
├── GET /1inch_tokens?network=base (463ms)
├── GET /1inch_tokens?network=avax (463ms)
├── GET /1inch_tokens?network=zksync (467ms)
├── GET /1inch_tokens?network=optimism (663ms)
├── GET /1inch_tokens?network=polygon (463ms)
├── GET /1inch_tokens?network=bsc (700ms)
├── GET /coingecko/coins_markets?page=1 (212ms)
├── GET /coingecko/coins_markets?page=2 (524ms)
├── GET /coingecko/coins_markets?page=3 (259ms)
└── GET /api.binance.com/api/v3/ticker/price?symbol=WOOUSDT (93ms)

After Initial Load (t=3000ms+)
├── Periodic statistics updates
├── Network filter changes
├── Period filter changes
└── User interactions
```

## Key Features

### Real-Time Analytics

- **Live Statistics**: Updated periodically with latest trading data
- **Multi-Network Support**: Filter by network to see network-specific statistics
- **Time Period Filtering**: View statistics for different time periods
- **Source Breakdown**: See which aggregators are driving volume
- **Token-Level Analytics**: Detailed statistics per token

### Visualizations

- **Volume Chart**: Time-series chart of trading volume
- **Source Pie Chart**: Breakdown of volume by source aggregator
- **Token Table**: Sortable table of token statistics
- **Metrics Cards**: Key performance indicators (addresses, trades, turnover rate)

### Data Accuracy

- **Updated Timestamps**: All statistics include `updated_at` timestamps
- **Raw Units**: All volumes returned in raw units (requires division by 1e18)
- **Precise Calculations**: Turnover rates calculated with high precision
- **Cross-Network Aggregation**: Multi-total statistics aggregate across all networks

## Summary

WooFi's dashboard page uses:

1. **WooFi API** (`api.woofi.com`) for:
   - Daily statistics (`/stat`)
   - Source breakdown (`/source_stat`)
   - Token statistics (`/token_stat`)
   - Cumulative statistics (`/cumulate_stat`)
   - Network totals (`/total_stat`)
   - Cross-network totals (`/multi_total_stat`)

2. **Third-Party APIs** for:
   - Price data (Binance)
   - Token metadata (1inch via proxy)
   - Market data (CoinGecko via proxy)

3. **Data Flow**:
   - **GET requests**: Fetch statistics, source breakdown, token data, cumulative metrics, totals
   - **POST requests**: Analytics tracking only
   - **Real-time updates**: Periodic polling for statistics
   - **Network/Period filtering**: Dynamic API calls based on user selections

The page is primarily a **read-only analytics dashboard** that fetches and displays comprehensive trading statistics. All data is fetched via GET requests, with POST requests used only for analytics tracking. The dashboard provides real-time insights into platform performance across multiple networks and time periods.

