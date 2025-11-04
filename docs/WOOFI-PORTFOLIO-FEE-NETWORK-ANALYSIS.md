# WooFi Portfolio Fee Tier - Network Fetching & Posting Analysis

This document analyzes how [WooFi Pro's portfolio fee tier page](https://woofi.com/en/portfolio/fee) fetches and posts data, including API endpoints, request/response structures, and data flow patterns.

## Overview

WooFi Pro's fee tier page displays fee tier information based on trading volume and WOO token staking levels. The page shows:
- **Fee Tier Table**: 7 tiers with volume requirements and fee rates
- **User Fee Tier Information**: Current tier, 30-day trading volume, WOO stake, and fee rates
- **Action Buttons**: "Trade" and "Stake WOO" buttons to improve tier

## Key API Endpoints

### Base API URLs
- **Orderly Mainnet API**: `https://api.orderly.org/v1/`
- **Orderly Testnet API**: `https://testnet-api.orderly.org/v1/`
- **WooFi CDN**: `https://oss.orderly.network/` (network logos)

## Network Requests Breakdown

### 1. Initial Page Load Requests

#### Standard Portfolio API Calls (Parallel)

The fee tier page makes the same standard API calls as other portfolio pages:

**Public Information:**
```
GET /v1/public/token
GET /v1/public/info
GET /v1/public/system_info
GET /v1/public/rwa/info
GET /v1/public/funding_rates
GET /v1/public/futures
GET /v1/public/leverage
GET /v1/public/chain_info?broker_id=woofi_pro
```

**System Status:**
```
GET /v1/public/system_info?source=maintenance
GET /v1/restricted_areas
GET /v1/ip_info
```

**Announcements:**
```
GET /v2/public/announcement
```

### 2. Fee Tier Data Structure

The fee tier table is **static/hardcoded** and displays the following structure:

**Fee Tier Table:**

| Tier | 30 Day Volume (USDC) | OR | WOO Staking Level | Maker / RWA Maker | Taker / RWA Taker |
|------|---------------------|-----|------------------|-------------------|-------------------|
| 1 | 0 - 500K | / | -- | 0.025% / 0% | 0.04% / 0.05% |
| 2 | 500K - 1M | / | level 1 - 2 (1.8K - 15.1K WOO) | 0.02% / 0% | 0.035% / 0.05% |
| 3 | 1M - 5M | / | level 3 - 4 (15.1K - 83K WOO) | 0.018% / 0% | 0.03% / 0.05% |
| 4 | 5M - 20M | / | level 5 (83K - 189.6K WOO) | 0.012% / 0% | 0.025% / 0.05% |
| 5 | 20M - 50M | / | level 6 (189.6K - 430.7K WOO) | 0.009% / 0% | 0.022% / 0.05% |
| 6 | 50M - 100M | / | level 7 - 8 (430.7K - 2209.7K WOO) | 0.005% / 0% | 0.018% / 0.05% |
| 7 | Above 100M | / | level 9 - 10 (Above 2209.7K WOO) | 0% / 0% | 0.015% / 0.05% |

**Key Observations:**
- Fee tiers are determined by **either** 30-day trading volume **OR** WOO staking level
- Higher tiers have lower maker/taker fees
- RWA (Real World Assets) trading has different fee rates
- Tier 7 (highest) has 0% maker fee for regular trading

### 3. User-Specific Fee Tier Information

When a user is **authenticated and connected**, the page displays user-specific information:

**User Fee Tier Display:**
- **Your tier**: Current fee tier number (1-7)
- **30D trading volume (USDC)**: User's 30-day trading volume
- **My stake (WOO)**: Amount of WOO tokens staked
- **Taker fee rate**: Current taker fee rate (with RWA rate)
- **Maker fee rate**: Current maker fee rate (with RWA rate)

**Note**: When not logged in, these fields show "--" or "-" placeholders.

### 4. Authenticated Endpoints (When User is Logged In)

User-specific fee tier data is likely fetched through authenticated endpoints:

**Potential Endpoints (Inferred):**
```
GET /v1/client/info
GET /v1/client/fee_tier
GET /v1/client/volume_30d
```

**Note**: These endpoints require authentication headers and would only be called when a user has connected their wallet.

## Data Flow Patterns

### Initial Page Load Flow

```
1. Page Load
   ↓
2. Standard Portfolio API Calls (Parallel):
   - GET /v1/public/token
   - GET /v1/public/info
   - GET /v1/public/system_info
   - GET /v1/public/chain_info?broker_id=woofi_pro
   - GET /v1/public/leverage
   ↓
3. System Status Checks:
   - GET /v1/public/system_info?source=maintenance
   - GET /v1/restricted_areas
   - GET /v1/ip_info
   ↓
4. Announcements:
   - GET /v2/public/announcement
   ↓
5. Render Fee Tier Table (Static/Hardcoded)
   ↓
6. If User is Authenticated:
   - Fetch user fee tier information
   - Display user-specific data
   Else:
   - Display placeholder values ("--", "-")
```

### User Authentication Flow

```
1. User Connects Wallet
   ↓
2. Authentication Token Retrieved
   ↓
3. Fetch User Fee Tier Data:
   - GET /v1/client/info (includes fee tier)
   - GET /v1/client/volume_30d (30-day trading volume)
   - GET /v1/client/staking (WOO stake amount)
   ↓
4. Update UI with User Data:
   - Display current tier
   - Display 30D trading volume
   - Display WOO stake amount
   - Display current fee rates
```

## Fee Tier Structure Details

### Tier Determination

Users can qualify for a fee tier through **two methods**:

1. **Trading Volume Method**: Based on 30-day trading volume in USDC
   - Tier 1: $0 - $500K
   - Tier 2: $500K - $1M
   - Tier 3: $1M - $5M
   - Tier 4: $5M - $20M
   - Tier 5: $20M - $50M
   - Tier 6: $50M - $100M
   - Tier 7: Above $100M

2. **WOO Staking Method**: Based on WOO token staking level
   - Tier 1: No staking requirement
   - Tier 2: Level 1-2 (1.8K - 15.1K WOO)
   - Tier 3: Level 3-4 (15.1K - 83K WOO)
   - Tier 4: Level 5 (83K - 189.6K WOO)
   - Tier 5: Level 6 (189.6K - 430.7K WOO)
   - Tier 6: Level 7-8 (430.7K - 2209.7K WOO)
   - Tier 7: Level 9-10 (Above 2209.7K WOO)

**User's tier is determined by the HIGHER of the two methods.**

### Fee Rate Structure

**Regular Trading Fees:**
- Maker fees: Range from 0.025% (Tier 1) to 0% (Tier 7)
- Taker fees: Range from 0.04% (Tier 1) to 0.015% (Tier 7)

**RWA (Real World Assets) Trading Fees:**
- Maker fees: 0% across all tiers
- Taker fees: 0.05% across all tiers

### Fee Tier Benefits

**Tier Progression Benefits:**
- **Lower Fees**: Each tier offers reduced maker and taker fees
- **Volume-Based**: Traders with high volume can access lower fees
- **Staking Alternative**: Users can stake WOO tokens instead of trading volume
- **Best of Both**: System uses the higher tier from volume or staking

## UI Features

### Fee Tier Display

**Header Section:**
- **Title**: "Fee tier"
- **Update Notice**: "Updated daily by ~2:15 UTC"

**User Information Card:**
- **Your tier**: Current tier number
- **30D trading volume (USDC)**: Last 30 days trading volume
- **My stake (WOO)**: Current WOO staking amount
- **Taker fee rate**: Current taker fee (with RWA rate)
- **Maker fee rate**: Current maker fee (with RWA rate)

**Fee Tier Table:**
- 7 rows showing all available tiers
- Columns: Tier, 30-day volume, WOO staking level, Maker/RWA Maker fees, Taker/RWA Taker fees
- Responsive design for mobile/desktop

**Action Buttons:**
- **"Trade" button**: Navigates to trading page to increase trading volume
- **"Stake WOO" button**: Navigates to WOO staking page to increase staking level

## Comparison with temp-clone Implementation

### temp-clone Version
- Uses `FeeTierModule.FeeTierPage` from `@orderly.network/portfolio` package
- Same static fee tier table structure
- User-specific data fetched through portfolio package's internal APIs
- Limited customization options

### WooFi Pro Version (woofi.com/en/portfolio/fee)
- Uses the same `FeeTierModule.FeeTierPage` component from `@orderly.network/portfolio`
- **Same fee tier structure** (7 tiers)
- **Same fee rates** (maker/taker, regular/RWA)
- **Same user information display** (tier, volume, stake, rates)
- **Same action buttons** (Trade, Stake WOO)
- Integrated into WooFi Pro's portfolio section

**Key Difference**: The WooFi Pro version is integrated into the live production site with full wallet connection and user authentication support.

## Fee Tier Calculation

### How User Tier is Determined

1. **Calculate Volume-Based Tier:**
   - Fetch user's 30-day trading volume
   - Match volume to tier range (Tier 1-7)

2. **Calculate Staking-Based Tier:**
   - Fetch user's WOO staking amount
   - Match staking level to tier (Tier 1-7)

3. **Select Higher Tier:**
   - System uses the **higher tier** from the two methods
   - Example: If volume qualifies for Tier 3, but staking qualifies for Tier 5, user gets Tier 5

4. **Apply Fee Rates:**
   - Use the selected tier's fee rates
   - Apply maker/taker fees based on order type
   - Apply RWA fees for Real World Asset trades

### Fee Tier Updates

**Update Frequency:**
- Fee tiers are **updated daily at ~2:15 UTC**
- 30-day trading volume is calculated on a rolling basis
- WOO staking level is updated in real-time when stake changes

**Effective Date:**
- Tier changes take effect after the daily update
- Users can see their projected tier based on current volume/stake

## API Request/Response Details

### Standard Public Endpoints

**Token Information:**
```
GET /v1/public/token
```

**Response**: List of supported tokens with metadata

**System Information:**
```
GET /v1/public/system_info
```

**Response**: System status, maintenance info, and configuration

**Chain Information:**
```
GET /v1/public/chain_info?broker_id=woofi_pro
```

**Response**: Supported blockchain networks and chain IDs

**Leverage Information:**
```
GET /v1/public/leverage
```

**Response**: Available leverage options for trading

### User-Specific Endpoints (Authenticated)

**User Info (Inferred):**
```
GET /v1/client/info
Headers:
  - Authorization: Bearer {token}
```

**Expected Response (Inferred):**
```json
{
  "success": true,
  "data": {
    "fee_tier": 3,
    "30d_volume": 2500000.50,
    "woo_stake": 50000.0,
    "maker_fee_rate": 0.018,
    "taker_fee_rate": 0.03,
    "rwa_maker_fee_rate": 0.0,
    "rwa_taker_fee_rate": 0.05
  }
}
```

**30-Day Volume (Inferred):**
```
GET /v1/client/volume_30d
Headers:
  - Authorization: Bearer {token}
```

**Expected Response (Inferred):**
```json
{
  "success": true,
  "data": {
    "volume_usdc": 2500000.50,
    "start_date": "2025-10-05",
    "end_date": "2025-11-04"
  }
}
```

**WOO Staking (Inferred):**
```
GET /v1/client/staking
Headers:
  - Authorization: Bearer {token}
```

**Expected Response (Inferred):**
```json
{
  "success": true,
  "data": {
    "woo_staked": 50000.0,
    "staking_level": 4,
    "next_level_threshold": 83000.0
  }
}
```

## Summary

WooFi Pro's fee tier page displays:

1. **Static Fee Tier Table**: 7 tiers with volume/staking requirements and fee rates
2. **User-Specific Information**: Current tier, trading volume, WOO stake, and fee rates (when authenticated)
3. **Action Buttons**: "Trade" and "Stake WOO" to improve tier
4. **Daily Updates**: Fee tiers updated daily at ~2:15 UTC

The page uses the `@orderly.network/portfolio` package's `FeeTierModule.FeeTierPage` component, which handles:
- Display of fee tier table
- User information display
- Fee rate calculations
- Integration with wallet connection

User-specific data requires authentication and is fetched through authenticated endpoints when a wallet is connected. The fee tier structure is static/hardcoded and matches the standard Orderly Network fee tier system.

