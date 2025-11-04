# API Data Fetching Analysis - Orderly Network DEX

This document explains how the codebase fetches symbols, trading data, recent trades, and trade records from Orderly Network's API.

## Overview

The data fetching is primarily handled by **Orderly Network SDK packages** (`@orderly.network/react-app`, `@orderly.network/trading`, etc.), which abstract away the direct API calls. However, we can identify the API endpoints and data flow by examining the codebase and network requests.

## Base API URLs

The Orderly Network API base URLs are determined by the `networkId`:

- **Mainnet**: `https://api.orderly.org` or `https://api.orderly.network`
- **Testnet**: `https://testnet-api.orderly.org` or `https://testnet-api.orderly.network`

The `networkId` is set in:
- **File**: `app/components/orderlyProvider/index.tsx`
- **Function**: `getNetworkId()` - reads from localStorage key `"orderly_network_id"` or defaults to `"mainnet"`

## 1. Symbols List Fetching

### How It Works

The symbol list is fetched automatically by the Orderly Network SDK when `OrderlyAppProvider` initializes.

### API Endpoints Used

#### Primary Endpoint (Build Time)
**File**: `build.ts` (lines 29-38)
```typescript
async function fetchSymbols(): Promise<string[]> {
  const response = await fetch("https://api.orderly.org/v1/public/info");
  const data = await response.json();
  return data.data.rows.map((row) => row.symbol);
}
```

**Endpoint**: `GET https://api.orderly.org/v1/public/info`
- **Purpose**: Fetches all available symbols (used during build for static route generation)
- **Response**: `{ success: boolean, data: { rows: SymbolInfo[] } }`
- **Note**: This doesn't include `broker_id` parameter, so it returns all symbols

#### Runtime Endpoints (SDK Internal)
Based on network requests observed, the SDK internally calls:

**For Symbol List**:
- `GET https://api.orderly.org/v1/public/info?broker_id={brokerId}` (mainnet)
- `GET https://testnet-api.orderly.org/v1/public/info?broker_id={brokerId}` (testnet)

**For Chain Info**:
- `GET https://api.orderly.org/v1/public/chain_info?broker_id={brokerId}` (mainnet)
- `GET https://testnet-api.orderly.org/v1/public/chain_info?broker_id={brokerId}` (testnet)

**Configuration**:
- `brokerId` comes from: `VITE_ORDERLY_BROKER_ID` in `public/config.js`
- `networkId` comes from: localStorage `"orderly_network_id"` or default `"mainnet"`

### Code Flow

```
OrderlyAppProvider (app/components/orderlyProvider/index.tsx)
    ↓
Receives: brokerId, networkId
    ↓
SDK Internally Calls: /v1/public/info?broker_id={brokerId}
    ↓
Filters symbols based on broker configuration
    ↓
Provides symbol list to TradingPage component
```

### Symbol Format

Symbols follow: `{TYPE}_{BASE}_{QUOTE}`
- Example: `PERP_ETH_USDC`
  - `PERP` = Perpetual contract type
  - `ETH` = Base asset
  - `USDC` = Quote asset

## 2. Trading Data Fetching

### How It Works

Trading data (prices, orderbook, market info) is fetched by the `TradingPage` component from `@orderly.network/trading` package.

### API Endpoints Used

#### Symbol Information
- `GET https://api.orderly.org/v1/public/info/{SYMBOL}` (mainnet)
- `GET https://testnet-api.orderly.org/v1/public/info/{SYMBOL}` (testnet)
  - **Example**: `GET https://api.orderly.org/v1/public/info/PERP_ETH_USDC`
  - **Purpose**: Get detailed information about a specific symbol (price, 24h change, volume, etc.)

#### Futures Market Data
- `GET https://api.orderly.org/v1/public/futures` (mainnet)
- `GET https://testnet-api.orderly.org/v1/public/futures` (testnet)
  - **Purpose**: Get list of all perpetual futures contracts

#### Specific Symbol Futures Data
- `GET https://api.orderly.org/v1/public/futures/{SYMBOL}` (mainnet)
- `GET https://testnet-api.orderly.org/v1/public/futures/PERP_ETH_USDC` (testnet)
  - **Purpose**: Get detailed futures contract information for a specific symbol

#### Funding Rates
- `GET https://api.orderly.org/v1/public/funding_rates` (mainnet)
- `GET https://testnet-api.orderly.org/v1/public/funding_rates` (testnet)
  - **Purpose**: Get current funding rates for all symbols

#### Specific Symbol Funding Rate
- `GET https://api.orderly.org/v1/public/funding_rate/{SYMBOL}` (mainnet)
- `GET https://testnet-api.orderly.org/v1/public/funding_rate/PERP_ETH_USDC` (testnet)
  - **Purpose**: Get funding rate for a specific symbol

#### System Information
- `GET https://api.orderly.org/v1/public/system_info` (mainnet)
- `GET https://testnet-api.orderly.org/v1/public/system_info` (testnet)
  - **Purpose**: Get system-wide information and status

#### Token Information
- `GET https://api.orderly.org/v1/public/token` (mainnet)
- `GET https://testnet-api.orderly.org/v1/public/token` (testnet)
  - **Purpose**: Get list of supported tokens

#### Leverage Information
- `GET https://api.orderly.org/v1/public/leverage` (mainnet)
- `GET https://testnet-api.orderly.org/v1/public/leverage` (testnet)
  - **Purpose**: Get available leverage options

### Code Flow

```
TradingPage Component (app/pages/perp/Symbol.tsx)
    ↓
Receives: symbol prop (e.g., "PERP_ETH_USDC")
    ↓
SDK Internally Fetches:
  - /v1/public/info/{SYMBOL} (symbol details)
  - /v1/public/futures/{SYMBOL} (futures contract info)
  - /v1/public/funding_rate/{SYMBOL} (funding rate)
  - TradingView datafeed for chart history
    ↓
Displays: Price, 24h change, volume, orderbook, chart, etc.
```

## 3. Recent Trades Fetching

### How It Works

Recent trades are fetched using the `useLastTradesScript` hook from `@orderly.network/trading` package.

### Component Usage

**File**: `node_modules/@orderly.network/trading/dist/index.d.ts` (lines 186-193)

```typescript
declare const useLastTradesScript: (symbol: string) => {
  base: string;
  quote: string;
  data: API.Trade[];          // Array of recent trades
  isLoading: boolean;
  baseDp: number;             // Base asset decimal places
  quoteDp: number;            // Quote asset decimal places
};
```

This hook is used by the `LastTradesWidget` component, which is rendered within `TradingPage`.

### API Endpoints Used

Based on typical Orderly Network API structure:

- **Recent Trades**:
  - `GET https://api.orderly.org/v1/public/market_trades?symbol={SYMBOL}&limit={LIMIT}` (mainnet)
  - `GET https://testnet-api.orderly.org/v1/public/market_trades?symbol={SYMBOL}&limit={LIMIT}` (testnet)
  - **Example**: `GET https://api.orderly.org/v1/public/market_trades?symbol=PERP_ETH_USDC&limit=100`

**Note**: The exact endpoint path may vary. The SDK abstracts this, so we need to check the actual network requests to confirm.

### TradingView Datafeed Integration

For chart historical data, TradingView uses its own datafeed configured in:

**File**: `app/utils/config.tsx`
```typescript
tradingViewConfig: {
  scriptSRC: withBasePath("/tradingview/charting_library/charting_library.js"),
  library_path: withBasePath("/tradingview/charting_library/"),
  // ...
}
```

TradingView datafeed likely calls:
- `GET https://api.orderly.org/tv/history?symbol={SYMBOL}&resolution={RESOLUTION}&from={FROM}&to={TO}&countback={COUNTBACK}`
  - **Example**: `GET https://api.orderly.org/tv/history?symbol=PERP_ETH_USDC&resolution=15&from=1761896883&to=1762192983&countback=329`
- `GET https://api.orderly.org/tv/config`
- `GET https://api.orderly.org/tv/symbol_info?group=Orderly`

### Code Flow

```
TradingPage Component
    ↓
Contains: LastTradesWidget (from @orderly.network/trading)
    ↓
Uses: useLastTradesScript(symbol) hook
    ↓
SDK Internally Calls: /v1/public/market_trades?symbol={SYMBOL}
    ↓
Returns: Array of recent trades with price, quantity, timestamp
```

## 4. Trade Records (Testnet & Mainnet)

### How It Works

Trade records (user's trading history) are fetched by portfolio-related components from `@orderly.network/portfolio` package.

### Components

**File**: `app/pages/portfolio/History.tsx`
```typescript
import { HistoryModule } from "@orderly.network/portfolio";
// Uses: <HistoryModule.HistoryPage />
```

**File**: `app/pages/portfolio/Positions.tsx`
```typescript
import { PositionsModule } from "@orderly.network/portfolio";
// Uses: <PositionsModule.PositionsPage />
```

### API Endpoints Used

#### User Trade History (RESTRICTED - Requires Authentication)
These endpoints require authentication headers with API keys:

- `GET https://api.orderly.org/v1/orders` (mainnet)
- `GET https://testnet-api.orderly.org/v1/orders` (testnet)
  - **Purpose**: Get user's order history
  - **Auth**: Required (API key + signature)

- `GET https://api.orderly.org/v1/positions` (mainnet)
- `GET https://testnet-api.orderly.org/v1/positions` (testnet)
  - **Purpose**: Get user's current positions
  - **Auth**: Required

- `GET https://api.orderly.org/v1/execution` (mainnet)
- `GET https://testnet-api.orderly.org/v1/execution` (testnet)
  - **Purpose**: Get user's execution/trade history
  - **Auth**: Required

- `GET https://api.orderly.org/v1/positions/info` (mainnet)
- `GET https://testnet-api.orderly.org/v1/positions/info` (testnet)
  - **Purpose**: Get aggregated position information
  - **Auth**: Required

### Network Switching

The network (mainnet/testnet) is determined by:

1. **Environment Variables**:
   - `VITE_DISABLE_MAINNET` - If true, forces testnet
   - `VITE_DISABLE_TESTNET` - If true, forces mainnet

2. **localStorage**:
   - Key: `"orderly_network_id"`
   - Values: `"mainnet"` or `"testnet"`

3. **User Chain Selection**:
   - When user connects a wallet and switches chains
   - `onChainChanged` callback in `app/components/orderlyProvider/index.tsx` detects if chain is testnet or mainnet
   - Updates `networkId` and reloads the page

### Demo Broker Events (Special Case)

**File**: `app/hooks/useDemoBrokerEvents.ts`

This hook fetches trading events for demo broker accounts:

```typescript
const response = await fetch(
  "https://orderly-dashboard-query-service.orderly.network/events_v2",
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      account_id: accountId,
      from_time: thirtyDaysAgo,
      to_time: now,
    }),
  }
);
```

**Endpoint**: `POST https://orderly-dashboard-query-service.orderly.network/events_v2`
- **Purpose**: Get trading events for demo broker accounts (last 30 days)
- **Method**: POST
- **Body**: `{ account_id: string, from_time: number, to_time: number }`

### Code Flow for Trade Records

```
User Connects Wallet
    ↓
OrderlyAppProvider detects networkId (mainnet/testnet)
    ↓
Portfolio Components (History, Positions, Orders)
    ↓
SDK Authenticates using user's API key (if configured)
    ↓
SDK Calls:
  - /v1/orders (mainnet/testnet based on networkId)
  - /v1/positions (mainnet/testnet)
  - /v1/execution (mainnet/testnet)
    ↓
Displays: User's trading history, positions, orders
```

## 5. API Endpoint Summary

### Public Endpoints (No Authentication Required)

| Endpoint | Purpose | Network |
|----------|---------|---------|
| `/v1/public/info` | Get all symbols list | Both |
| `/v1/public/info/{SYMBOL}` | Get specific symbol info | Both |
| `/v1/public/futures` | Get all futures contracts | Both |
| `/v1/public/futures/{SYMBOL}` | Get specific futures contract | Both |
| `/v1/public/funding_rates` | Get all funding rates | Both |
| `/v1/public/funding_rate/{SYMBOL}` | Get specific funding rate | Both |
| `/v1/public/system_info` | Get system information | Both |
| `/v1/public/token` | Get supported tokens | Both |
| `/v1/public/leverage` | Get leverage options | Both |
| `/v1/public/chain_info` | Get chain information | Both |
| `/v1/public/market_trades` | Get recent market trades | Both |
| `/v1/ip_info` | Get user IP and region | Both |
| `/tv/config` | TradingView config | Both |
| `/tv/history` | TradingView historical data | Both |
| `/tv/symbol_info` | TradingView symbol info | Both |

### Restricted Endpoints (Authentication Required)

| Endpoint | Purpose | Network |
|----------|---------|---------|
| `/v1/orders` | Get user's orders | Both |
| `/v1/positions` | Get user's positions | Both |
| `/v1/execution` | Get user's trade executions | Both |
| `/v1/positions/info` | Get aggregated position info | Both |

## 6. Network ID Detection & Switching

### How Network ID is Determined

**File**: `app/components/orderlyProvider/index.tsx` (lines 16-31)

```typescript
const getNetworkId = (): NetworkId => {
  // 1. Check if mainnet/testnet is disabled via config
  const disableMainnet = getRuntimeConfigBoolean('VITE_DISABLE_MAINNET');
  const disableTestnet = getRuntimeConfigBoolean('VITE_DISABLE_TESTNET');
  
  if (disableMainnet && !disableTestnet) return "testnet";
  if (disableTestnet && !disableMainnet) return "mainnet";
  
  // 2. Check localStorage
  return localStorage.getItem("orderly_network_id") || "mainnet";
};
```

### Network Switching

When a user connects a wallet and changes chains:

```typescript
const onChainChanged = useCallback(
  (_chainId: number, {isTestnet}: {isTestnet: boolean}) => {
    const currentNetworkId = getNetworkId();
    if ((isTestnet && currentNetworkId === 'mainnet') || 
        (!isTestnet && currentNetworkId === 'testnet')) {
      const newNetworkId: NetworkId = isTestnet ? 'testnet' : 'mainnet';
      setNetworkId(newNetworkId);
      setTimeout(() => {
        window.location.reload();  // Reload to switch networks
      }, 100);
    }
  },
  []
);
```

## 7. Broker ID Parameter

The `broker_id` query parameter is added to many API calls:

- **Source**: `VITE_ORDERLY_BROKER_ID` from `public/config.js`
- **Example URLs**:
  - `https://api.orderly.org/v1/public/chain_info?broker_id=woofi_pro`
  - `https://api.orderly.org/v1/public/info?broker_id=demo`
  - `https://testnet-api.orderly.org/v1/public/chain_info?broker_id=raydium`

The broker ID determines:
- Which symbols are available
- Broker-specific configurations
- Symbol filtering and visibility

## 8. Actual Network Requests Observed

From browser network inspection, here are the actual endpoints being called:

### Initialization (App Load)
1. `GET https://api.orderly.org/v1/public/system_info`
2. `GET https://api.orderly.org/v1/public/token`
3. `GET https://api.orderly.org/v1/public/info`
4. `GET https://api.orderly.org/v1/public/funding_rates`
5. `GET https://api.orderly.org/v1/public/futures`
6. `GET https://api.orderly.org/v1/public/chain_info?broker_id={brokerId}`
7. `GET https://api.orderly.org/v1/public/leverage`

### Trading Page Load (Specific Symbol)
1. `GET https://api.orderly.org/v1/public/futures/{SYMBOL}`
   - Example: `GET https://api.orderly.org/v1/public/futures/PERP_ETH_USDC`
2. `GET https://api.orderly.org/v1/public/info/{SYMBOL}`
   - Example: `GET https://api.orderly.org/v1/public/info/PERP_ETH_USDC`
3. `GET https://api.orderly.org/v1/public/funding_rate/{SYMBOL}`
   - Example: `GET https://api.orderly.org/v1/public/funding_rate/PERP_ETH_USDC`
4. `GET https://api.orderly.org/tv/config`
5. `GET https://api.orderly.org/tv/symbol_info?group=Orderly`
6. `GET https://api.orderly.org/tv/history?symbol={SYMBOL}&resolution={RES}&from={FROM}&to={TO}&countback={COUNT}`
   - Example: `GET https://api.orderly.org/tv/history?symbol=PERP_ETH_USDC&resolution=15&from=1761896883&to=1762192983&countback=329`

### Recent Trades (Real-time)
The `LastTradesWidget` component likely uses WebSocket connections or polling:
- WebSocket: `wss://api.orderly.org/ws` (for real-time trades)
- Or polling: `GET https://api.orderly.org/v1/public/market_trades?symbol={SYMBOL}`

## 9. SDK Abstraction Layers

The actual API calls are abstracted by these SDK packages:

### `@orderly.network/react-app`
- Provides `OrderlyAppProvider` component
- Manages API client initialization
- Handles authentication
- Provides context and hooks for accessing data

### `@orderly.network/trading`
- Provides `TradingPage` component
- Fetches symbol list, market data, trades
- Manages real-time data subscriptions
- Provides hooks like `useLastTradesScript`

### `@orderly.network/portfolio`
- Provides portfolio-related components
- Fetches user's positions, orders, trade history
- Handles authenticated API calls

## 10. Key Findings

### What We Can See in the Codebase

✅ **Visible in code**:
- `build.ts` - Fetches symbols at build time: `https://api.orderly.org/v1/public/info`
- `useDemoBrokerEvents.ts` - Fetches demo events: `https://orderly-dashboard-query-service.orderly.network/events_v2`
- `useIpRestriction.ts` - Fetches IP info: `https://api.orderly.org/v1/ip_info`
- Network ID detection logic
- Broker ID configuration

### What's Hidden in SDK

❌ **Hidden in SDK packages**:
- Exact API endpoint paths for trades (may use WebSocket)
- Authentication flow and signature generation
- Real-time subscription management
- Error handling and retry logic
- Rate limiting

### To See Actual API Calls

To see exactly what endpoints are being called:

1. **Browser DevTools**:
   - Open Network tab
   - Filter by "api.orderly" or "orderly.org"
   - Watch requests as you use the app

2. **Network Request Examples**:
   From our earlier inspection:
   ```
   GET https://api.orderly.org/v1/public/chain_info?broker_id=woofi_pro
   GET https://api.orderly.org/v1/public/futures/PERP_ETH_USDC
   GET https://api.orderly.org/tv/history?symbol=PERP_ETH_USDC&resolution=15&from=...&to=...
   ```

## 11. Configuration Impact

### Environment Variables

| Variable | Impact on API Calls |
|----------|-------------------|
| `VITE_ORDERLY_BROKER_ID` | Added as `?broker_id={value}` query parameter |
| `VITE_ORDERLY_NETWORK_ID` | Determines base URL (mainnet vs testnet) |
| `VITE_DISABLE_MAINNET` | Forces testnet API URLs |
| `VITE_DISABLE_TESTNET` | Forces mainnet API URLs |

### Runtime Configuration

**File**: `public/config.js`

All `VITE_*` variables are exposed here and read via:
- **File**: `app/utils/runtime-config.ts`
- **Function**: `getRuntimeConfig(key: string)`

## 12. Trade Records Access

### For Authenticated Users

Trade records are accessed through portfolio modules:

1. **Positions**: `PositionsModule.PositionsPage`
   - Fetches: `/v1/positions` (authenticated)

2. **Orders**: `OrdersModule` components
   - Fetches: `/v1/orders` (authenticated)

3. **History**: `HistoryModule.HistoryPage`
   - Fetches: `/v1/execution` or `/v1/orders` (authenticated)

4. **Assets**: `AssetsModule` components
   - Fetches: `/v1/client/info`, `/v1/asset/settlement_history` (authenticated)

### Authentication Flow

1. User connects wallet
2. SDK generates API key (or uses existing)
3. SDK signs requests with API key + secret
4. Requests include headers:
   - `ORDERLY-KEY`: API key
   - `ORDERLY-SIGN`: Signature
   - `ORDERLY-TIMESTAMP`: Timestamp

The authentication logic is handled internally by the SDK.

## Summary

### Symbols List
- **Build Time**: `GET https://api.orderly.org/v1/public/info` (in `build.ts`)
- **Runtime**: SDK calls `/v1/public/info?broker_id={brokerId}` based on `networkId`

### Trading Data
- **Symbol Info**: `/v1/public/info/{SYMBOL}`
- **Futures Data**: `/v1/public/futures/{SYMBOL}`
- **Funding Rate**: `/v1/public/funding_rate/{SYMBOL}`
- **Chart History**: `/tv/history?symbol={SYMBOL}&...`

### Recent Trades
- **Hook**: `useLastTradesScript(symbol)` from `@orderly.network/trading`
- **Endpoint**: Likely `/v1/public/market_trades?symbol={SYMBOL}` or WebSocket
- **Component**: `LastTradesWidget` or `LastTrades` in TradingPage

### Trade Records (User History)
- **Orders**: `/v1/orders` (authenticated)
- **Positions**: `/v1/positions` (authenticated)
- **Executions**: `/v1/execution` (authenticated)
- **Components**: `HistoryModule`, `PositionsModule`, `OrdersModule` from `@orderly.network/portfolio`

### Network Selection
- **Mainnet**: `https://api.orderly.org`
- **Testnet**: `https://testnet-api.orderly.org`
- **Determined by**: localStorage `"orderly_network_id"` or config flags

### Broker ID
- **Source**: `VITE_ORDERLY_BROKER_ID` from `public/config.js`
- **Usage**: Added as query parameter: `?broker_id={brokerId}`
- **Impact**: Filters symbols and configurations specific to that broker

