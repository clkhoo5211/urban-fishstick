# Markets Logic - Mainnet & Testnet

This document explains how the markets functionality works in both mainnet and testnet environments, including market listings, favorites management, and navigation.

## Overview

The markets functionality is implemented using the **Markets** package (`@orderly.network/markets`), which provides a comprehensive market browsing and discovery system. The markets page is primarily a **display and navigation interface** that shows available trading markets, market data, funding rates, and allows users to manage favorites/watchlists. It serves as a gateway to the trading interface.

## Architecture

### Component Structure

```
App.tsx
  └── OrderlyProvider (app/components/orderlyProvider/index.tsx)
      └── OrderlyAppProvider (from @orderly.network/react-app)
          └── MarketsLayout (app/pages/markets/Layout.tsx)
              └── Scaffold (from @orderly.network/ui-scaffold)
                  └── MarketsIndex (app/pages/markets/Index.tsx)
                      └── MarketsHomePage (from @orderly.network/markets)
                          ├── MarketsHeader
                          ├── MarketsDataList
                          ├── MarketsList
                          └── Funding tabs
```

### Key Files

1. **Markets Page Entry**: `app/pages/markets/Index.tsx`
   - Renders `<MarketsHomePage />` from `@orderly.network/markets`
   - Configures comparison props (exchange name, logo)

2. **Markets Layout**: `app/pages/markets/Layout.tsx`
   - Wraps markets pages with `Scaffold` component
   - Provides navigation structure

3. **Markets Package**: `node_modules/@orderly.network/markets/`
   - Contains all market display logic, UI components, and data fetching
   - Handles favorites management, search, sorting, and navigation

## Network Selection Logic

### How Network ID is Determined

The markets page uses the same network selection logic as other Orderly components:

**File**: `app/components/orderlyProvider/index.tsx`

```typescript
const getNetworkId = (): NetworkId => {
  if (typeof window === "undefined") return "mainnet";
  
  // 1. Check runtime config flags
  const disableMainnet = getRuntimeConfigBoolean('VITE_DISABLE_MAINNET');
  const disableTestnet = getRuntimeConfigBoolean('VITE_DISABLE_TESTNET');
  
  // 2. Force network based on flags
  if (disableMainnet && !disableTestnet) {
    return "testnet";
  }
  
  if (disableTestnet && !disableMainnet) {
    return "mainnet";
  }
  
  // 3. Check localStorage for user preference
  return (localStorage.getItem("orderly_network_id") as NetworkId) || "mainnet";
};
```

### Network Configuration Flow

```
User Action / Config
    ↓
getNetworkId() determines networkId
    ↓
OrderlyAppProvider receives networkId prop
    ↓
OrderlyAppProvider sets up API client with correct base URL:
    - Mainnet: https://api.orderly.org
    - Testnet: https://testnet-api.orderly.org
    ↓
MarketsHomePage component (from @orderly.network/markets) 
inherits network context from OrderlyAppProvider
    ↓
All market API calls use the correct base URL
```

### Runtime Configuration

**File**: `public/config.js`

```javascript
{
  "VITE_DISABLE_MAINNET": "false",  // If "true", forces testnet
  "VITE_DISABLE_TESTNET": "false",  // If "true", forces mainnet
  "VITE_ORDERLY_BROKER_ID": "demo",
  "VITE_ORDERLY_BROKER_NAME": "Aeternus JinFinex",
  "VITE_HAS_SECONDARY_LOGO": "false",
  "VITE_ORDERLY_MAINNET_CHAINS": "1,56,43114,...",
  "VITE_ORDERLY_TESTNET_CHAINS": "97,421614,901901901,..."
}
```

## Markets Page Features

### 1. Market Listings

**Purpose**: Display all available trading markets

**Features**:
- List of all markets/symbols
- Market data display:
  - Market name (symbol)
  - Last price
  - 24h change (percentage and absolute)
  - 24h volume
  - 8h funding rate
- Sorting capabilities:
  - By price
  - By 24h change
  - By 24h volume
  - By funding rate
- Pagination support
- Real-time updates

### 2. Market Tabs

**Available Tabs**:

1. **Favorites**
   - User's favorite markets
   - Multiple favorite lists (watchlists)
   - Custom sorting per list
   - Pin to top functionality

2. **Recent**
   - Recently viewed markets
   - Automatically tracked
   - Quick access to recent symbols

3. **All Markets**
   - Complete list of all available markets
   - Full market data
   - Sortable columns

4. **New Listings**
   - Recently added markets
   - Highlight new symbols
   - Special tagging

5. **RWA** (Real World Assets)
   - RWA markets category
   - Filtered view

### 3. Market Header

**Features**:
- Market statistics:
  - Total 24h volume
  - Total open interest
  - TVL (Total Value Locked)
- Top gainers
- Top losers
- News feed (if available)
- Market overview

### 4. Search Functionality

**Features**:
- Search markets by symbol name
- Real-time filtering
- Search placeholder: "Search market"
- Case-insensitive search

### 5. Favorites Management

**Features**:
- Add/remove favorites
- Multiple favorite lists (watchlists)
- Maximum 10 favorite lists
- List name up to 15 characters
- Custom sorting per list
- Pin symbols to top
- Dropdown menu for managing favorites

**Operations**:
- Add symbol to favorites
- Remove symbol from favorites
- Create new favorite list
- Delete favorite list
- Rename favorite list
- Move symbol to different list
- Pin symbol to top

### 6. Funding Rate Information

**Funding Tab**:
- Overview: All markets with funding rates
- Comparison: Compare funding rates across exchanges
- Funding rate data:
  - Estimated funding rate
  - Last funding rate
  - 1D, 3D, 7D, 14D, 30D, 90D averages
  - Positive rate percentage

## Markets API Endpoints

Based on the type definitions and typical Orderly Network API structure, the markets operations use the following endpoints:

### Mainnet Endpoints

#### Market Data
- **Get All Symbols**: `GET https://api.orderly.org/v1/public/info?broker_id={brokerId}`
- **Get Symbol Info**: `GET https://api.orderly.org/v1/public/info/{SYMBOL}`
- **Get Futures List**: `GET https://api.orderly.org/v1/public/futures`
- **Get Futures Info**: `GET https://api.orderly.org/v1/public/futures/{SYMBOL}`
- **Get Funding Rates**: `GET https://api.orderly.org/v1/public/funding_rates`
- **Get Funding Rate**: `GET https://api.orderly.org/v1/public/funding_rate/{SYMBOL}`
- **Get Market Trades**: `GET https://api.orderly.org/v1/public/market_trades?symbol={SYMBOL}`

#### Market Statistics
- **Get System Info**: `GET https://api.orderly.org/v1/public/system_info`
- **Get Daily PnL**: `GET https://api.orderly.org/v1/public/daily_pnl` (if available)

### Testnet Endpoints

All endpoints use the testnet base URL: `https://testnet-api.orderly.org`

**Note**: The exact endpoint paths may vary. These are inferred from the API structure and typical market patterns.

## Submission Operations

The markets page is primarily a **display and navigation interface**. The only "submissions" are:

### 1. Favorites Management (Local Storage)

#### Add to Favorites

**Step-by-Step Process**:

1. **User Clicks Favorite Icon**
   - User clicks star/favorite icon next to a market
   - Opens favorites dropdown menu

2. **User Selects Favorite List**
   - User selects existing favorite list
   - OR creates new favorite list
   - Confirms selection

3. **Update Local Storage**
   - Symbol added to selected favorite list
   - Stored in browser localStorage
   - UI updates immediately

**Data Storage**:
- Stored in localStorage (browser)
- Key: Typically `orderly_favorites` or similar
- Format: JSON structure with favorite lists and symbols

#### Remove from Favorites

1. **User Clicks Favorite Icon** (already favorited)
2. **User Unchecks Favorite List**
3. **Update Local Storage**
   - Symbol removed from favorite list
   - Stored in localStorage
   - UI updates immediately

#### Create Favorite List

1. **User Opens Favorites Menu**
2. **User Clicks "Add New List"**
3. **User Enters List Name**
   - Name: Maximum 15 characters
   - Validates: Unique name, not empty
4. **User Confirms**
   - New list created
   - Stored in localStorage
   - Maximum 10 lists (enforced)

#### Delete Favorite List

1. **User Selects Favorite List**
2. **User Clicks Delete**
3. **Confirmation Dialog**
   - Shows: "Are you sure you want to delete {name}?"
4. **User Confirms**
   - List deleted from localStorage
   - UI updates

#### Pin Symbol to Top

1. **User Clicks on Symbol Menu**
2. **User Selects "Move to Top"**
3. **Update Local Storage**
   - Symbol order updated
   - Pinned symbol moves to top of list
   - Stored in localStorage

### 2. Navigation to Trading Page

**Step-by-Step Process**:

1. **User Clicks on Market**
   - User clicks on any market row/symbol
   - OR clicks on symbol in header/info bar

2. **Navigation Triggered**
   - `onSymbolChange` callback called
   - Navigates to `/perp/{SYMBOL}`
   - Preserves query parameters if any

3. **Trading Page Loads**
   - Trading page receives symbol
   - Loads trading interface for that symbol
   - User can start trading

**No API Submission**: This is just navigation, no API calls required.

## Network-Specific Behavior

### Mainnet Behavior

- **Base URL**: `https://api.orderly.org`
- **Real Market Data**: Real prices, volumes, funding rates
- **Production Markets**: All production markets available
- **Real Statistics**: Actual 24h volume, open interest, TVL
- **Production Symbols**: Production trading symbols

### Testnet Behavior

- **Base URL**: `https://testnet-api.orderly.org`
- **Test Market Data**: Testnet prices, volumes, funding rates
- **Test Markets**: Testnet markets only
- **Test Statistics**: Testnet 24h volume, open interest, TVL
- **Test Symbols**: Testnet trading symbols

### Network Detection

The markets components receive the network context through:

1. **OrderlyAppProvider Context**: Provides `networkId` to all child components
2. **API Client**: Automatically uses correct base URL based on `networkId`
3. **Broker ID**: Added to all API requests from `VITE_ORDERLY_BROKER_ID`

## Key Features

### Market Data Display

1. **Real-Time Updates**
   - Prices update in real-time
   - Volume updates continuously
   - Funding rates update periodically

2. **Sorting**
   - Sort by any column
   - Ascending/descending order
   - Maintains sort state per tab

3. **Filtering**
   - Search by symbol name
   - Filter by market category
   - Filter by funding rate

4. **Pagination**
   - Paginated market lists
   - Configurable page size
   - Page navigation

### Favorites System

1. **Multiple Lists**
   - Create up to 10 favorite lists
   - Custom names (max 15 chars)
   - Independent sorting per list

2. **Symbol Management**
   - Add/remove symbols
   - Move between lists
   - Pin to top
   - Track recent symbols

3. **Local Storage**
   - Persisted in browser
   - User-specific
   - No server synchronization

### Funding Rate Information

1. **Funding Overview**
   - All markets with funding rates
   - Historical averages
   - Positive rate statistics

2. **Funding Comparison**
   - Compare with other exchanges
   - Exchange name and logo configurable
   - Side-by-side comparison

## Key Differences: Mainnet vs Testnet

| Aspect | Mainnet | Testnet |
|--------|---------|---------|
| **API Base URL** | `https://api.orderly.org` | `https://testnet-api.orderly.org` |
| **Market Data** | Real prices/volumes | Testnet prices/volumes |
| **Markets** | Production markets | Testnet markets |
| **Symbols** | Production symbols | Testnet symbols |
| **Funding Rates** | Real funding rates | Test funding rates |
| **Statistics** | Real 24h volume/OI/TVL | Test statistics |
| **Broker ID** | Production broker | Test broker |
| **Network ID** | `"mainnet"` | `"testnet"` |

## Important Notes

1. **Display Only**: The markets page is primarily for browsing and navigation. It doesn't submit trading orders or transactions.

2. **Favorites Local Only**: Favorites are stored in browser localStorage and are not synchronized across devices or sessions on the server.

3. **Real-Time Data**: Market data (prices, volumes) updates in real-time via WebSocket or polling.

4. **Broker-Specific**: Markets shown are filtered by `broker_id`. Different brokers may have different available markets.

5. **Navigation Gateway**: The main purpose is to help users discover and navigate to trading pages.

6. **No Authentication Required**: Market data is public and doesn't require authentication. However, favorites are user-specific (localStorage).

7. **Network Isolation**: Mainnet and testnet show completely different markets and data.

8. **Symbol Format**: Symbols follow format: `{TYPE}_{BASE}_{QUOTE}` (e.g., `PERP_ETH_USDC`).

## Example: Complete Favorite Management Flow

### Mainnet

```
1. User Opens Markets Page
   - networkId = "mainnet"
   - API Base URL = "https://api.orderly.org"
   - Shows all mainnet markets

2. User Clicks Favorite Icon on PERP_ETH_USDC
   - Opens favorites dropdown
   - Shows existing favorite lists

3. User Creates New List
   - Clicks "Add New List"
   - Enters name: "My Favorites"
   - Confirms creation

4. User Adds Symbol
   - Selects "My Favorites" list
   - PERP_ETH_USDC added to list
   - Stored in localStorage

5. User Views Favorites Tab
   - Switches to "Favorites" tab
   - Sees "My Favorites" list
   - PERP_ETH_USDC displayed

6. User Pins Symbol
   - Clicks on PERP_ETH_USDC menu
   - Selects "Move to Top"
   - Symbol moves to top of list
   - Updated in localStorage
```

## Example: Complete Navigation Flow

### Testnet

```
1. User Browses Markets
   - networkId = "testnet"
   - API Base URL = "https://testnet-api.orderly.org"
   - Views testnet markets

2. User Searches for Market
   - Types "BTC" in search
   - Filters to BTC markets
   - Sees PERP_BTC_USDC

3. User Clicks on PERP_BTC_USDC
   - onSymbolChange callback triggered
   - Navigates to /perp/PERP_BTC_USDC

4. Trading Page Loads
   - Trading page receives symbol
   - Loads market data for PERP_BTC_USDC
   - User can start trading

5. Symbol Added to Recent
   - PERP_BTC_USDC added to recent list
   - Stored in localStorage
   - Appears in "Recent" tab
```

## Debugging

### Check Current Network

```javascript
// In browser console
localStorage.getItem("orderly_network_id")  // "mainnet" or "testnet"
```

### Check Network Configuration

```javascript
// In browser console
window.__RUNTIME_CONFIG__
// Check:
// - VITE_DISABLE_MAINNET
// - VITE_DISABLE_TESTNET
// - VITE_ORDERLY_BROKER_ID
// - VITE_ORDERLY_BROKER_NAME
```

### Check Favorites Storage

```javascript
// In browser console
localStorage.getItem("orderly_favorites")  // JSON string of favorites
// Or check all localStorage keys
Object.keys(localStorage).filter(key => key.includes('favorite'))
```

### Monitor API Calls

1. Open browser DevTools → Network tab
2. Filter by "orderly" or "public"
3. Look for:
   - `GET /v1/public/info?broker_id={brokerId}` - Fetch all markets
   - `GET /v1/public/info/{SYMBOL}` - Fetch specific market info
   - `GET /v1/public/futures` - Fetch futures list
   - `GET /v1/public/funding_rates` - Fetch funding rates
   - `GET /v1/public/market_trades` - Fetch recent trades

### Check Market Data

The markets page displays:
- Symbol information
- Prices and changes
- Volumes
- Funding rates
- Market statistics

All data is fetched from public endpoints and doesn't require authentication.

## Summary

The markets functionality provides:

1. **Market Discovery**: Browse all available trading markets
2. **Market Data**: Real-time prices, volumes, funding rates
3. **Favorites Management**: Create watchlists and manage favorites (localStorage)
4. **Search & Filter**: Find markets quickly
5. **Sorting**: Sort markets by various criteria
6. **Navigation**: Navigate to trading pages
7. **Funding Information**: View and compare funding rates
8. **Network-Aware**: Uses correct API base URL based on `networkId`

The markets page is primarily a **display and navigation interface** with minimal submissions (only favorites management via localStorage). Most operations are display-only, and the page serves as a gateway to the trading interface. Users can browse markets, manage favorites locally, and navigate to trading pages for specific symbols. All market data is fetched from public API endpoints and works identically on both mainnet and testnet, with the only difference being the API base URL and which markets are available.

