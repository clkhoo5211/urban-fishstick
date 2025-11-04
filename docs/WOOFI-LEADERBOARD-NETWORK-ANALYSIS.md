# WooFi Pro Leaderboard - Network Fetching & Posting Analysis

This document analyzes how [WooFi Pro's leaderboard page](https://woofi.com/en/leaderboard) fetches and posts data, including all API endpoints, request/response structures, and data flow patterns. This is **different from the temp-clone leaderboard** which only uses the `@orderly.network/trading-leaderboard` package.

## Overview

WooFi Pro's leaderboard page is a comprehensive trading competition and ranking system that displays:
- **General Leaderboard**: Overall trading volume and PnL rankings across all users
- **Campaign-Specific Leaderboards**: Multiple trading competitions with their own rankings, prize pools, and rules
- **Campaign History**: Previous ended campaigns that users can view
- **Real-time Statistics**: Trading volume, participant counts, and prize pool information

## Key Differences from temp-clone Leaderboard

### temp-clone Version
- Uses `@orderly.network/trading-leaderboard` package (`GeneralLeaderboardWidget`)
- Single leaderboard view (general only)
- Campaign integration through the package's internal APIs
- No campaign history or campaign selection UI
- Limited customization

### WooFi Pro Version (woofi.com/en/leaderboard)
- **Custom implementation** with direct Orderly API calls
- **Multiple leaderboard types**: General + Campaign-specific
- **Campaign carousel**: Shows 5+ previous campaigns with images
- **Campaign detail pages**: Full campaign info, rules, terms, timelines
- **Advanced filtering**: Date ranges (7D, 14D, 30D, 90D), custom date picker
- **Search functionality**: Search by wallet address
- **Campaign images**: Custom campaign banners and rule images
- **Broker-specific**: Uses `broker_id=woofi_pro` for filtering

## Key API Endpoints

### Base API URLs
- **Orderly Mainnet API**: `https://api.orderly.org/v1/`
- **Orderly Testnet API**: `https://testnet-api.orderly.org/v1/`
- **WooFi CDN**: `https://woofi.com/leaderboard/` (campaign images)
- **Orderly CDN**: `https://oss.orderly.network/` (network logos)

## Network Requests Breakdown

### 1. Initial Page Load Requests

#### GET Requests (Data Fetching)

**General Leaderboard Data:**
```
GET /v1/broker/leaderboard/daily
  ?page=1
  &size=100
  &aggregateBy=address_per_builder
  &broker_id=woofi_pro
  &sort=descending_perp_volume
  &start_date=2025-08-07
  &end_date=2025-11-04
```

**Response Structure:**
```json
{
  "success": true,
  "data": {
    "rows": [
      {
        "perp_volume": 425675089.85586,
        "perp_taker_volume": 255272819.797828,
        "perp_maker_volume": 170402270.058032,
        "total_fee": 48445.488257,
        "broker_fee": 20957.101479,
        "address": "0x1f1690074a8744e8d38cdca1ab2b75fafe5d6d07",
        "broker_id": "woofi_pro",
        "realized_pnl": -1552856.06194
      }
      // ... more rows
    ]
  }
}
```

**Campaign Ranking (Default Campaign):**
```
GET /v1/public/campaign/ranking
  ?campaign_id=128
  &page=1
  &size=2000
  &aggregate_by=address
```

**Response Structure:**
```json
{
  "success": true,
  "data": {
    "rows": [
      {
        "volume": 177822012.56707,
        "address": "0x1f1690074a8744e8d38cdca1ab2b75fafe5d6d07",
        "pnl": -506435.530092,
        "total_deposit_amount": 795099.513664,
        "total_withdrawal_amount": 0.0,
        "start_account_value": 1308309.323621,
        "end_account_value": 1596973.307193,
        "roi": -0.2408
      }
      // ... more rows
    ]
  }
}
```

**Campaign Stats (General):**
```
GET /v1/public/campaign/stats
  ?campaign_id=
  &symbols=
  &broker_id=woofi_pro
  &group_by=BROKER
```

**Campaign Stats (Specific Campaign):**
```
GET /v1/public/campaign/stats
  ?campaign_id=128
  &symbols=
  &broker_id=woofi_pro
  &group_by=BROKER
```

**Response Structure:**
```json
{
  "success": true,
  "data": {
    "volume": 362826019.54811,
    "user_count": 246,
    "updated_time": 1760918430081
  },
  "timestamp": 1762229845893
}
```

**Campaign Assets:**
- Campaign images: `https://woofi.com/leaderboard/campaign_{campaign_id}.png`
- Campaign rule images: `https://woofi.com/leaderboard/rule_{campaign_id}_{metric}.png`
  - Example: `rule_127_volume.png`, `rule_127_pnl.png`

### 2. Campaign-Specific Requests

When a user clicks on a campaign card, the following requests are made:

**Campaign Ranking (with sorting):**
```
GET /v1/public/campaign/ranking
  ?page=1
  &size=100
  &campaign_id=127
  &broker_id=woofi_pro
  &sort_by=volume
```

**Alternative sorting options:**
- `sort_by=volume` - Sort by trading volume (default)
- `sort_by=pnl` - Sort by profit and loss
- `sort_by=roi` - Sort by return on investment

**Campaign Stats:**
```
GET /v1/public/campaign/stats
  ?campaign_id=127
  &symbols=
  &broker_id=woofi_pro
  &group_by=BROKER
```

**Response includes:**
- Total trading volume for the campaign
- Number of participants
- Last updated timestamp
- Prize pool information (if available)

### 3. Date Range Filtering

The general leaderboard supports date range filtering:

**Date Range Parameters:**
- `start_date`: YYYY-MM-DD format (e.g., `2025-08-07`)
- `end_date`: YYYY-MM-DD format (e.g., `2025-11-04`)
- Quick filters: `7D`, `14D`, `30D`, `90D` (calculated from current date)

**Example with 7D filter:**
```
GET /v1/broker/leaderboard/daily
  ?page=1
  &size=100
  &aggregateBy=address_per_builder
  &broker_id=woofi_pro
  &sort=descending_perp_volume
  &start_date=2025-10-28
  &end_date=2025-11-04
```

### 4. Search Functionality

The page includes a search input that filters leaderboard results by wallet address:

**Client-side filtering:**
- Search is performed on the already-loaded leaderboard data
- Filters addresses that match the search query (partial match)
- Supports both EVM addresses (0x...) and Solana addresses

**Example:**
- User types: `0x1f16`
- Results filtered to show addresses containing `0x1f16`

### 5. Pagination

**Pagination Parameters:**
- `page`: Page number (default: 1)
- `size`: Number of results per page (default: 100, options: 25, 50, 100)

**Example:**
```
GET /v1/broker/leaderboard/daily
  ?page=2
  &size=50
  &aggregateBy=address_per_builder
  &broker_id=woofi_pro
  &sort=descending_perp_volume
  &start_date=2025-08-07
  &end_date=2025-11-04
```

## Data Flow Patterns

### General Leaderboard Flow

```
1. Page Load
   ↓
2. Fetch General Leaderboard Data
   GET /v1/broker/leaderboard/daily
   ↓
3. Fetch Campaign Stats (General)
   GET /v1/public/campaign/stats?campaign_id=&broker_id=woofi_pro
   ↓
4. Load Campaign Images
   GET /leaderboard/campaign_{id}.png (for each campaign)
   ↓
5. Render Leaderboard Table
   - Rank, Address, Trading Volume, PnL
   ↓
6. User Interactions:
   - Date range filter → Update start_date/end_date → Re-fetch
   - Search address → Client-side filter
   - Pagination → Update page parameter → Re-fetch
```

### Campaign-Specific Flow

```
1. User Clicks Campaign Card
   ↓
2. Fetch Campaign Details
   GET /v1/public/campaign/stats?campaign_id={id}
   ↓
3. Fetch Campaign Ranking
   GET /v1/public/campaign/ranking?campaign_id={id}&sort_by=volume
   ↓
4. Load Campaign Assets
   GET /leaderboard/rule_{id}_volume.png
   GET /leaderboard/rule_{id}_pnl.png
   ↓
5. Render Campaign Detail Page
   - Campaign info (dates, title, description)
   - Prize pool information
   - Trading volume and participant stats
   - Campaign timeline
   - Rules and Terms sections
   - Leaderboard table (Trading volume or PnL tab)
   ↓
6. User Tabs Switch
   - "Trading volume" tab → sort_by=volume
   - "PnL" tab → sort_by=pnl
   → Re-fetch with new sort_by parameter
```

## API Request/Response Details

### General Leaderboard Endpoint

**Endpoint:** `GET /v1/broker/leaderboard/daily`

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | number | No | Page number (default: 1) |
| `size` | number | No | Results per page (default: 100) |
| `aggregateBy` | string | Yes | Aggregation method: `address_per_builder` |
| `broker_id` | string | Yes | Broker identifier: `woofi_pro` |
| `sort` | string | Yes | Sort order: `descending_perp_volume` |
| `start_date` | string | Yes | Start date: `YYYY-MM-DD` |
| `end_date` | string | Yes | End date: `YYYY-MM-DD` |

**Response Fields:**
- `perp_volume`: Total perpetual trading volume
- `perp_taker_volume`: Taker volume
- `perp_maker_volume`: Maker volume
- `total_fee`: Total fees paid
- `broker_fee`: Broker portion of fees
- `address`: Wallet address (EVM or Solana)
- `broker_id`: Broker identifier
- `realized_pnl`: Realized profit and loss

### Campaign Ranking Endpoint

**Endpoint:** `GET /v1/public/campaign/ranking`

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `campaign_id` | number | Yes | Campaign ID (e.g., 128, 127) |
| `page` | number | No | Page number (default: 1) |
| `size` | number | No | Results per page (default: 100, max: 2000) |
| `aggregate_by` | string | No | Aggregation: `address` |
| `broker_id` | string | No | Broker identifier |
| `sort_by` | string | No | Sort field: `volume`, `pnl`, `roi` |

**Response Fields:**
- `volume`: Trading volume for the campaign
- `address`: Wallet address
- `pnl`: Profit and loss
- `total_deposit_amount`: Total deposits during campaign
- `total_withdrawal_amount`: Total withdrawals during campaign
- `start_account_value`: Account value at campaign start
- `end_account_value`: Account value at campaign end
- `roi`: Return on investment (decimal)

### Campaign Stats Endpoint

**Endpoint:** `GET /v1/public/campaign/stats`

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `campaign_id` | number | No | Campaign ID (empty for general stats) |
| `symbols` | string | No | Symbol filter (usually empty) |
| `broker_id` | string | Yes | Broker identifier: `woofi_pro` |
| `group_by` | string | Yes | Grouping: `BROKER` |

**Response Fields:**
- `volume`: Total trading volume
- `user_count`: Number of participants
- `updated_time`: Last update timestamp (milliseconds)

## Campaign Information

### Campaign Structure

Each campaign includes:

1. **Campaign Metadata:**
   - Campaign ID (e.g., 128, 127, 121, 120, 116)
   - Campaign name (e.g., "Ascension", "The Haven Trading Competition")
   - Campaign tag (e.g., "THEHAVEN", "KOREA")
   - Status: "Ended" or "Active"
   - Prize pool amount (e.g., "104K", "20K", "5K")

2. **Campaign Timeline:**
   - Registration start/end dates
   - Battle start date
   - Battle end date
   - All dates in GMT+8 timezone

3. **Campaign Rules:**
   - Eligibility requirements (e.g., referral link required)
   - Trading volume requirements
   - Prize distribution rules
   - Ranking criteria (volume-based or PnL-based)
   - Reward currency (usually USDC)

4. **Prize Pool:**
   - Base reward amount
   - Dynamic bonus conditions (e.g., if volume reaches $500M, bonus increases)
   - Tiered rewards (top 20 users on each leaderboard)

5. **Campaign Assets:**
   - Campaign banner image: `/leaderboard/campaign_{id}.png`
   - Rule images: `/leaderboard/rule_{id}_{metric}.png`

### Example Campaigns

**Campaign 128 (Ascension):**
- Status: Ended
- Prize Pool: 104K USDC
- Image: `campaign_128.png`

**Campaign 127 (The Haven Trading Competition):**
- Status: Ended
- Prize Pool: 20K USDC
- Tag: THEHAVEN
- Dates: Sep 11, 2025 - Oct 3, 2025
- Eligibility: Only users referred by THEHAVEN
- Dynamic bonus: 20K base → 40K if volume reaches $500M

**Campaign 121 (Fireant x WOOFi Pro Trading Celebration):**
- Status: Ended
- Prize Pool: 5K USDC
- Tag: KOREA

**Campaign 120 (RECRUIT & REIGN):**
- Status: Ended
- Prize Pool: 6K USDC

**Campaign 116 (DAWN OF DOMINANCE):**
- Status: Ended
- Prize Pool: 25K USDC

## Network Configuration

### Network Detection

The page uses the same network detection logic as other WooFi pages:
- Detects mainnet vs testnet based on wallet connection
- Uses Orderly API endpoints accordingly:
  - Mainnet: `https://api.orderly.org/v1/`
  - Testnet: `https://testnet-api.orderly.org/v1/`

### Broker ID

All requests include `broker_id=woofi_pro` to filter results specific to WooFi Pro:
- Ensures rankings only show WooFi Pro users
- Campaign stats are broker-specific
- Leaderboard data is broker-specific

## UI Features

### Leaderboard Table

**Columns:**
1. **Rank**: Numerical rank (1st, 2nd, 3rd, etc.) with badges for top 3
2. **Address**: Wallet address (truncated display, clickable to Orderly dashboard)
3. **Trading Volume**: Total trading volume in USD
4. **PnL**: Realized profit and loss in USD

**Features:**
- Sortable columns (Trading Volume, PnL)
- Pagination (100 rows per page by default)
- Address search/filter
- Responsive design for mobile/desktop

### Campaign Carousel

**Features:**
- Horizontal scrollable carousel
- Shows 5 campaigns at a time
- Previous/Next navigation buttons
- Campaign cards show:
  - Status badge ("Ended")
  - Prize pool amount
  - Campaign name
  - Campaign image

### Campaign Detail Page

**Sections:**
1. **Campaign Header:**
   - Campaign title
   - Date range
   - Description

2. **Prize Pool Section:**
   - Current total prize pool
   - Trading volume (if active)
   - Participant count
   - Dynamic bonus conditions

3. **Campaign Timeline:**
   - Registration period
   - Battle start
   - Battle end
   - Visual timeline display

4. **Leaderboard Table:**
   - Tabs: "Trading volume" and "PnL"
   - Rank, Address, Volume/PnL columns
   - Same pagination and search features

5. **Rules Section:**
   - Campaign rules
   - Key notes
   - Eligibility requirements
   - Reward distribution rules

6. **Terms and Conditions:**
   - Legal terms
   - Participation requirements
   - Risk disclaimers

## Comparison with temp-clone Implementation

### temp-clone Leaderboard
- **Package**: `@orderly.network/trading-leaderboard`
- **Component**: `GeneralLeaderboardWidget`
- **API Calls**: Handled internally by the package
- **Features**: Basic leaderboard display only
- **Campaigns**: Limited campaign integration through package
- **Customization**: Limited to package props

### WooFi Pro Leaderboard
- **Implementation**: Custom React components
- **API Calls**: Direct Orderly API calls
- **Features**: 
  - General leaderboard
  - Campaign-specific leaderboards
  - Campaign history
  - Campaign detail pages
  - Advanced filtering
  - Search functionality
- **Campaigns**: Full campaign management with images, rules, terms
- **Customization**: Complete control over UI/UX

## Summary

WooFi Pro's leaderboard page is a comprehensive trading competition platform that goes far beyond the basic leaderboard in the temp-clone. It provides:

1. **Multiple Leaderboard Types**: General + Campaign-specific
2. **Campaign Management**: Full campaign lifecycle with history, details, rules, and terms
3. **Advanced Filtering**: Date ranges, search, pagination
4. **Rich UI**: Campaign images, rule images, timelines, prize pools
5. **Broker-Specific**: All data filtered by `broker_id=woofi_pro`

The implementation uses direct Orderly API calls rather than a pre-built package, giving full control over the user experience and allowing for extensive customization and feature additions.

