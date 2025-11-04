# Leaderboard Logic - Mainnet & Testnet

This document explains how the leaderboard functionality works in both mainnet and testnet environments.

## Overview

The leaderboard functionality is implemented using the **Trading Leaderboard** package (`@orderly.network/trading-leaderboard` version 2.8.1), which provides a comprehensive leaderboard system that displays trader rankings based on trading volume, realized PnL (Profit and Loss), and campaign participation. Unlike vaults and swaps, the leaderboard is **read-only** - it displays data but doesn't handle submissions. Users automatically appear on the leaderboard by trading on the platform.

## Architecture

### Component Structure

```
App.tsx
  └── OrderlyProvider (app/components/orderlyProvider/index.tsx)
      └── OrderlyAppProvider (from @orderly.network/react-app)
          └── LeaderboardLayout (app/pages/leaderboard/Layout.tsx)
              └── LeaderboardIndex (app/pages/leaderboard/Index.tsx)
                  └── GeneralLeaderboardWidget (from @orderly.network/trading-leaderboard)
```

### Key Files

1. **Leaderboard Page Entry**: `app/pages/leaderboard/Index.tsx`
   - Renders `<GeneralLeaderboardWidget />` component
   - Provides SEO tags

2. **Leaderboard Layout**: `app/pages/leaderboard/Layout.tsx`
   - Provides Scaffold navigation structure
   - Sets initial menu to `/leaderboard`

3. **Leaderboard Package**: `node_modules/@orderly.network/trading-leaderboard/`
   - Contains all leaderboard logic, UI components, and API calls
   - Handles data fetching, ranking, and display

## Network Selection Logic

### How Network ID is Determined

The leaderboard uses the same network selection logic as other Orderly components:

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
GeneralLeaderboardWidget component (from @orderly.network/trading-leaderboard) 
inherits network context from OrderlyAppProvider
    ↓
All leaderboard API calls use the correct base URL
    ↓
Leaderboard data is fetched based on networkId
```

### Runtime Configuration

**File**: `public/config.js`

```javascript
{
  "VITE_DISABLE_MAINNET": "false",  // If "true", forces testnet
  "VITE_DISABLE_TESTNET": "false",  // If "true", forces mainnet
  "VITE_ORDERLY_BROKER_ID": "demo",
  "VITE_ORDERLY_MAINNET_CHAINS": "1,56,43114,...",
  "VITE_ORDERLY_TESTNET_CHAINS": "97,421614,901901901,..."
}
```

## Leaderboard API Endpoints

Based on the type definitions and typical Orderly Network API structure, the leaderboard operations use the following endpoints:

### Mainnet Endpoints

- **Get General Leaderboard**: `GET https://api.orderly.org/v1/public/leaderboard`
- **Get Campaign Leaderboard**: `GET https://api.orderly.org/v1/public/leaderboard/campaign/{campaignId}`
- **Get Campaign Stats**: `GET https://api.orderly.org/v1/public/campaign/stats`
- **Get User Ranking**: `GET https://api.orderly.org/v1/public/leaderboard/user/{accountId}`

### Testnet Endpoints

- **Get General Leaderboard**: `GET https://testnet-api.orderly.org/v1/public/leaderboard`
- **Get Campaign Leaderboard**: `GET https://testnet-api.orderly.org/v1/public/leaderboard/campaign/{campaignId}`
- **Get Campaign Stats**: `GET https://testnet-api.orderly.org/v1/public/campaign/stats`
- **Get User Ranking**: `GET https://testnet-api.orderly.org/v1/public/leaderboard/user/{accountId}`

**Note**: The exact endpoint paths may vary. These are inferred from the API structure and typical leaderboard patterns.

### Query Parameters

Common query parameters for leaderboard endpoints:

- `broker_id`: Broker ID (from `VITE_ORDERLY_BROKER_ID`)
- `period`: Time period (7d, 14d, 30d, 90d, all_time)
- `metric`: Ranking metric (`volume` or `pnl`)
- `limit`: Number of results per page
- `offset`: Pagination offset
- `symbol`: Filter by trading symbol (optional)

## Leaderboard Data Structures

### TradingData Interface

```typescript
interface TradingData {
  account_id: string;           // User's account ID
  address: string;              // User's wallet address
  broker_fee: number;           // Broker fees paid
  date: string;                 // Date of the data
  perp_maker_volume: number;    // Perpetual maker volume
  perp_taker_volume: number;    // Perpetual taker volume
  perp_volume: number;          // Total perpetual volume
  total_fee: number;            // Total fees paid
  rank?: number;                // User's rank
  pnl?: number;                 // Realized PnL
  rewards?: number;             // Estimated rewards
}
```

### CampaignConfig Interface

```typescript
interface CampaignConfig {
  campaign_id: number | string;
  title: string;
  subtitle?: string;
  description: string;
  start_time: string;
  end_time: string;
  reward_distribution_time?: string;
  volume_scope?: string | string[];
  referral_codes?: string[] | string;
  prize_pools?: PrizePool[];
  tiered_prize_pools?: Array<PrizePool[]>;
  ticket_rules?: TicketRules;
  image?: string;
  rule_url?: string;
  trading_url?: string;
  leaderboard_config?: LeaderboardConfig;
}
```

### PrizePool Interface

```typescript
interface PrizePool {
  pool_id: string;              // Prize pool identifier
  label: string;                // Prize pool label
  total_prize: number;          // Total prize amount
  currency: string;             // Reward currency (USDC, etc.)
  metric: "volume" | "pnl";     // Evaluation metric
  tiers: PrizePoolTier[];       // Reward tiers
  volume_limit?: number;        // Volume limit for eligibility
}
```

### UserData Interface

```typescript
interface UserData {
  rank?: number | string;       // User's current rank
  pnl: number;                  // User's realized PnL
  total_participants?: number;  // Total participants
  volume: number;               // User's trading volume
  referral_code?: string;       // User's referral code
  estimated_rewards?: number;   // Estimated rewards
  estimated_tickets?: number;   // Estimated tickets earned
}
```

## How Users Get on the Leaderboard

### Automatic Inclusion

Users **automatically** appear on the leaderboard by:

1. **Trading on the Platform**
   - Users don't need to submit anything
   - Simply trading (making orders, positions) qualifies them
   - Their trading activity is tracked automatically

2. **Data Aggregation**
   - Orderly Network aggregates trading data in real-time
   - Trading volume, PnL, and fees are calculated automatically
   - Rankings are updated based on selected metrics

3. **Broker Scope**
   - Leaderboard is filtered by `broker_id`
   - Only traders on the same broker appear together
   - Each broker has their own leaderboard

### Ranking Criteria

Users are ranked based on:

1. **Trading Volume** (default)
   - Total perpetual trading volume
   - Includes both maker and taker volume
   - Calculated over selected time period

2. **Realized PnL**
   - Total realized profit and loss
   - Based on closed positions
   - Can be positive or negative

3. **Campaign-Specific Metrics**
   - Some campaigns may use different metrics
   - Custom ranking rules per campaign
   - May include referral codes, specific symbols, etc.

## Leaderboard Features

### 1. General Leaderboard

The general leaderboard shows overall trader rankings:

- **Rankings**: Top traders by volume or PnL
- **Time Periods**: 7 days, 14 days, 30 days, 90 days, all time
- **Metrics**: Volume or PnL
- **Pagination**: Supports pagination for large lists
- **Search**: Search by wallet address
- **Sorting**: Sort by rank, volume, PnL, etc.

### 2. Campaign Leaderboards

Campaign-specific leaderboards for trading competitions:

- **Campaigns**: Ongoing, past, and future campaigns
- **Prize Pools**: Multiple prize pools per campaign
- **Rewards**: Estimated rewards based on current ranking
- **Tickets**: Ticket-based lotteries
- **Eligibility**: Volume requirements, referral codes, etc.

### 3. User Profile

Individual user information:

- **Current Rank**: User's position in leaderboard
- **Trading Volume**: Total volume in selected period
- **Realized PnL**: Total PnL
- **Estimated Rewards**: Potential rewards based on current rank
- **Estimated Tickets**: Tickets earned in campaigns

### 4. Filters and Sorting

- **Time Period**: Filter by 7d, 14d, 30d, 90d, all time
- **Metric**: Sort by volume or PnL
- **Search**: Search by wallet address or account ID
- **Pagination**: Navigate through pages
- **Real-time Updates**: Data refreshes automatically

## Data Flow

### Step-by-Step Process

1. **User Opens Leaderboard Page**
   - Navigates to `/leaderboard`
   - `LeaderboardIndex` component renders
   - `GeneralLeaderboardWidget` initializes

2. **Widget Initialization**
   - `GeneralLeaderboardWidget` reads network context from `OrderlyAppProvider`
   - Determines API base URL based on `networkId`
   - Sets up API client with correct base URL

3. **Data Fetching**
   - Widget fetches leaderboard data from API
   - Includes `broker_id` query parameter
   - Fetches for selected time period and metric

4. **Ranking Calculation**
   - API calculates rankings based on trading data
   - Sorts by selected metric (volume or PnL)
   - Assigns ranks to each trader

5. **Display**
   - Widget displays leaderboard table
   - Shows rank, address, volume, PnL, rewards
   - Highlights current user's position (if connected)

6. **Real-time Updates**
   - Widget periodically refreshes data
   - Updates rankings as new trades occur
   - Shows "Last update" timestamp

7. **User Interaction**
   - User can change time period
   - User can switch between volume and PnL metrics
   - User can search for specific addresses
   - User can navigate through pages

## Network-Specific Behavior

### Mainnet Behavior

- **Base URL**: `https://api.orderly.org`
- **Real Trading Data**: Shows actual trading volume and PnL from mainnet
- **Production Campaigns**: Displays active production campaigns
- **Real Rewards**: Prize pools with real rewards
- **Broker Filter**: Filtered by configured broker ID

### Testnet Behavior

- **Base URL**: `https://testnet-api.orderly.org`
- **Test Trading Data**: Shows trading data from testnet
- **Test Campaigns**: Displays test campaigns
- **Test Rewards**: Test prize pools (may not have real rewards)
- **Broker Filter**: Filtered by configured broker ID

### Network Detection

The leaderboard components receive the network context through:

1. **OrderlyAppProvider Context**: Provides `networkId` to all child components
2. **API Client**: Automatically uses correct base URL based on `networkId`
3. **Broker ID**: Added to all API requests from `VITE_ORDERLY_BROKER_ID`

## Campaign System

### Campaign Types

1. **Ongoing Campaigns**
   - Currently active trading competitions
   - Users can participate by trading
   - Real-time rankings and rewards

2. **Past Campaigns**
   - Completed campaigns
   - Final rankings and winners
   - Historical data

3. **Future Campaigns**
   - Upcoming campaigns
   - Registration information
   - Start time countdown

### Campaign Features

1. **Prize Pools**
   - Multiple prize pools per campaign
   - Fixed position rewards (1st, 2nd, 3rd)
   - Position range rewards (4th-10th, etc.)
   - Total prize amount displayed

2. **Ticket System**
   - Tiered mode: Different ticket amounts per volume tier
   - Linear mode: Earn tickets based on trading volume
   - Tickets for prize pool lottery

3. **Eligibility Rules**
   - Minimum trading volume requirements
   - Specific symbol requirements
   - Referral code requirements
   - Registration period

4. **Rewards Calculation**
   - Estimated rewards based on current rank
   - Real-time updates as rankings change
   - Final rewards at campaign end

## Key Differences: Mainnet vs Testnet

| Aspect | Mainnet | Testnet |
|--------|---------|---------|
| **API Base URL** | `https://api.orderly.org` | `https://testnet-api.orderly.org` |
| **Trading Data** | Real trading volume/PnL | Testnet trading data |
| **Campaigns** | Production campaigns | Test campaigns |
| **Rewards** | Real prize pools | Test prize pools |
| **Users** | Real traders | Test traders |
| **Broker ID** | Production broker | Test broker |
| **Network ID** | `"mainnet"` | `"testnet"` |

## Important Notes

1. **No Submission Required**: Users automatically appear on the leaderboard by trading. No manual submission or registration needed.

2. **Broker-Specific**: Each broker has their own leaderboard. Traders only see rankings within their broker's ecosystem.

3. **Real-time Updates**: Leaderboard data updates in real-time as trades occur. Rankings change dynamically.

4. **Network Isolation**: Mainnet and testnet leaderboards are completely separate. Data from one doesn't affect the other.

5. **Campaign Participation**: Users participate in campaigns automatically by trading. No separate registration needed (unless campaign requires it).

6. **Privacy**: Wallet addresses are displayed, but users can choose to display custom names if supported.

7. **Historical Data**: Leaderboard supports historical periods (7d, 14d, 30d, 90d, all time) for analyzing past performance.

8. **Multiple Metrics**: Leaderboard can be sorted by volume or PnL, providing different perspectives on trader performance.

9. **Rewards Estimation**: Estimated rewards are calculated based on current rank and may change as rankings update.

10. **Network Context**: Leaderboard automatically uses the correct network (mainnet/testnet) based on `OrderlyAppProvider` context.

## Example: Complete Leaderboard Flow

### Mainnet Leaderboard

```
1. User Opens Leaderboard
   - networkId = "mainnet"
   - API Base URL = "https://api.orderly.org"
   - brokerId = "demo"

2. Widget Initializes
   - GeneralLeaderboardWidget reads network context
   - Sets up API client with mainnet base URL
   - Default period: 30 days
   - Default metric: volume

3. Data Fetching
   - GET https://api.orderly.org/v1/public/leaderboard?broker_id=demo&period=30d&metric=volume
   - Returns list of top traders with:
     - Rank
     - Address
     - Trading volume
     - Realized PnL
     - Estimated rewards

4. Display
   - Widget displays leaderboard table
   - Shows top 100 traders (or paginated)
   - Highlights user's position if connected
   - Shows "Last update" timestamp

5. User Interactions
   - User changes period to 7 days
   - Widget fetches new data
   - Rankings update
   - User switches to PnL metric
   - Widget fetches PnL-based rankings
   - Display updates

6. Campaign View
   - User clicks on campaign
   - Widget fetches campaign-specific leaderboard
   - Shows campaign prize pools
   - Displays estimated rewards based on current rank
```

### Testnet Leaderboard

```
1. User Opens Leaderboard
   - networkId = "testnet"
   - API Base URL = "https://testnet-api.orderly.org"
   - brokerId = "demo"

2. Widget Initializes
   - Same initialization process
   - Uses testnet API base URL

3. Data Fetching
   - GET https://testnet-api.orderly.org/v1/public/leaderboard?broker_id=demo&period=30d&metric=volume
   - Returns testnet trading data

4. Display
   - Same display logic
   - Shows testnet traders and rankings

5. User Interactions
   - Same interaction capabilities
   - All features work identically on testnet
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
```

### Monitor API Calls

1. Open browser DevTools → Network tab
2. Filter by "orderly" or "leaderboard"
3. Look for:
   - `GET /v1/public/leaderboard` - General leaderboard
   - `GET /v1/public/leaderboard/campaign/{id}` - Campaign leaderboard
   - `GET /v1/public/campaign/stats` - Campaign statistics

### Check Leaderboard Data

The leaderboard data is fetched automatically by the widget. You can inspect:
- Network requests in browser DevTools
- React component state (if using React DevTools)
- API responses to see data structure

## Summary

The leaderboard functionality:

1. **No Submission Required**: Users automatically appear by trading
2. **Network-Aware**: Uses correct API base URL based on `networkId`
3. **Broker-Specific**: Filtered by `broker_id` configuration
4. **Real-time**: Updates as trading activity occurs
5. **Multiple Metrics**: Supports volume and PnL rankings
6. **Campaign Support**: Displays campaign-specific leaderboards
7. **Rewards Display**: Shows estimated rewards and tickets
8. **Historical Data**: Supports multiple time periods

The leaderboard is a **read-only display system** that automatically tracks and ranks traders based on their trading activity. Users don't need to submit anything - they simply appear on the leaderboard by trading on the platform. The system works identically on both mainnet and testnet, with the only difference being the API base URL and the data source.

