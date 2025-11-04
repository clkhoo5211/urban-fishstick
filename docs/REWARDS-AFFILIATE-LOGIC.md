# Rewards & Affiliate Logic - Mainnet & Testnet

This document explains how the rewards and affiliate referral system works in both mainnet and testnet environments.

## Overview

The rewards functionality is implemented using the **Affiliate** package (`@orderly.network/affiliate` version 2.8.1), which provides a comprehensive referral and affiliate program system. The rewards page redirects to the affiliate program, where users can either become affiliates (referrers) to earn commissions from referrals, or use referral codes as traders to receive rebates.

## Architecture

### Component Structure

```
App.tsx
  └── OrderlyProvider (app/components/orderlyProvider/index.tsx)
      └── OrderlyAppProvider (from @orderly.network/react-app)
          └── RewardsLayout (app/pages/rewards/Layout.tsx)
              └── RewardsIndex (app/pages/rewards/Index.tsx)
                  └── Redirects to /rewards/affiliate
                      └── RewardsAffiliate (app/pages/rewards/Affiliate.tsx)
                          └── ReferralProvider (from @orderly.network/affiliate)
                              └── Dashboard.AffiliatePage (from @orderly.network/affiliate)
```

### Key Files

1. **Rewards Page Entry**: `app/pages/rewards/Index.tsx`
   - Redirects to `/rewards/affiliate`
   - Preserves query parameters

2. **Affiliate Page**: `app/pages/rewards/Affiliate.tsx`
   - Renders `ReferralProvider` with configuration
   - Renders `Dashboard.AffiliatePage` component

3. **Affiliate Package**: `node_modules/@orderly.network/affiliate/`
   - Contains all affiliate/referral logic, UI components, and API calls
   - Handles referral code generation, binding, and commission tracking

## Network Selection Logic

### How Network ID is Determined

The affiliate/rewards system uses the same network selection logic as other Orderly components:

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
ReferralProvider component (from @orderly.network/affiliate) 
inherits network context from OrderlyAppProvider
    ↓
All affiliate/referral API calls use the correct base URL
```

### Runtime Configuration

**File**: `public/config.js`

```javascript
{
  "VITE_DISABLE_MAINNET": "false",  // If "true", forces testnet
  "VITE_DISABLE_TESTNET": "false",  // If "true", forces mainnet
  "VITE_ORDERLY_BROKER_ID": "demo",
  "VITE_ENABLE_CAMPAIGNS": "true"
}
```

## Affiliate API Endpoints

Based on the type definitions and typical Orderly Network API structure, the affiliate/referral operations use the following endpoints:

### Mainnet Endpoints

- **Get Referral Info**: `GET https://api.orderly.org/v1/referral/info`
- **Get Referral Statistics**: `GET https://api.orderly.org/v1/referral/statistics`
- **Get Daily Volume**: `GET https://api.orderly.org/v1/referral/daily_volume`
- **Bind Referral Code**: `POST https://api.orderly.org/v1/referral/bind`
- **Generate Referral Code**: `POST https://api.orderly.org/v1/referral/generate`
- **Update Referral Code**: `PUT https://api.orderly.org/v1/referral/code`
- **Update Referral Rate**: `PUT https://api.orderly.org/v1/referral/rate`
- **Get Referees List**: `GET https://api.orderly.org/v1/referral/referees`
- **Get Commission History**: `GET https://api.orderly.org/v1/referral/commission`

### Testnet Endpoints

- **Get Referral Info**: `GET https://testnet-api.orderly.org/v1/referral/info`
- **Get Referral Statistics**: `GET https://testnet-api.orderly.org/v1/referral/statistics`
- **Get Daily Volume**: `GET https://testnet-api.orderly.org/v1/referral/daily_volume`
- **Bind Referral Code**: `POST https://testnet-api.orderly.org/v1/referral/bind`
- **Generate Referral Code**: `POST https://testnet-api.orderly.org/v1/referral/generate`
- **Update Referral Code**: `PUT https://testnet-api.orderly.org/v1/referral/code`
- **Update Referral Rate**: `PUT https://testnet-api.orderly.org/v1/referral/rate`
- **Get Referees List**: `GET https://testnet-api.orderly.org/v1/referral/referees`
- **Get Commission History**: `GET https://testnet-api.orderly.org/v1/referral/commission`

**Note**: The exact endpoint paths may vary. These are inferred from the API structure and typical referral/affiliate patterns.

## Two Roles: Affiliate (Referrer) vs Trader (Referee)

### 1. Affiliate (Referrer)

**What it is**: Users who refer others and earn commissions

**How to become**:
- Trade $10,000+ on main account (auto-unlocks referral code)
- OR apply for affiliate status (higher commission rates)

**Benefits**:
- Earn up to 60% commission from referred traders' fees
- Default: 10% commission
- Can set commission sharing rate with referees
- Track referral statistics
- View commission history

**Features**:
- Generate/edit referral codes
- Share referral links
- View referees list
- Set commission sharing rate
- View commission earnings

### 2. Trader (Referee)

**What it is**: Users who use a referral code to get rebates

**How to participate**:
- Bind a referral code (before or after first trade)
- Trade using the referral code
- Receive rebates on trading fees

**Benefits**:
- Get rebates on trading fees
- Share of broker's net fee (minus Orderly fee)
- Rebates applied automatically

**Features**:
- Enter/bind referral code
- View rebate statistics
- View rebate history
- See who referred them

## Referral Code System

### Referral Code Format

- **Length**: 4-10 characters
- **Format**: Uppercase letters (A-Z) and numbers (0-9) only
- **Example**: `ABC123`, `REF2024`, `ORDERLY`

### Auto-Generation

When users meet requirements:
- **Requirement**: Trade $10,000+ on main account
- **Auto-generates**: Referral code automatically
- **Can customize**: Edit code (if not taken)
- **Can apply**: For higher commission rates via application

### Manual Generation

Users can:
- Generate referral code automatically (if eligible)
- Edit referral code (must be unique)
- Check code availability
- Use referral code immediately after generation

## Submission Flows

### 1. Bind Referral Code (Trader/Referee)

#### Step-by-Step Process

1. **User Opens Rewards Page**
   - Navigates to `/rewards`
   - Redirects to `/rewards/affiliate`
   - Sees "Trader" tab

2. **User Clicks "Enter Code"**
   - Opens referral code dialog
   - Shows input field for referral code

3. **User Enters Referral Code**
   - Enters 4-10 character code
   - Validates format (uppercase letters and numbers)
   - Checks if code exists

4. **Validation**
   - Format validation: Must be 4-10 characters, uppercase A-Z and 0-9
   - Existence check: Code must exist in system
   - Account check: Must be on main account (not sub-account)

5. **Request Submission**
   - POST request to `/v1/referral/bind`
   - Body:
     ```json
     {
       "referral_code": "ABC123"
     }
     ```
   - Includes authentication headers (API key + signature)

6. **Response Handling**
   - Success: Code bound successfully
   - Error: Code doesn't exist, already bound, etc.
   - Updates UI to show bound status

7. **Rebates Activation**
   - Rebates apply to future trades
   - User can view rebate statistics
   - Rebates tracked automatically

### 2. Generate Referral Code (Affiliate/Referrer)

#### Step-by-Step Process

1. **User Checks Eligibility**
   - Widget checks if user has traded $10,000+
   - OR checks if user has applied for affiliate status
   - Shows eligibility status

2. **User Clicks "Generate Code" or "Get Code"**
   - If auto-eligible: Code generated automatically
   - If needs application: Redirects to application form

3. **Auto-Generation**
   - POST request to `/v1/referral/generate`
   - Body:
     ```json
     {
       "auto_generate": true
     }
     ```
   - System generates unique code

4. **Code Assignment**
   - Code assigned to user
   - User can edit code (if desired)
   - Code ready to share immediately

5. **Edit Code (Optional)**
   - User can edit to custom code
   - PUT request to `/v1/referral/code`
   - Body:
     ```json
     {
       "referral_code": "CUSTOM123"
     }
     ```
   - Validates uniqueness
   - Updates if available

6. **Share Code**
   - User gets referral link
   - Format: `{referralLinkUrl}?referral_code={code}`
   - Can share via social media, copy link, etc.

### 3. Update Commission Rate (Affiliate)

#### Step-by-Step Process

1. **User Opens Settings**
   - Clicks on referral code settings
   - Opens commission rate dialog

2. **User Sets Sharing Rate**
   - Sees maximum commission rate (e.g., 60%)
   - Sets how much they receive vs referee receives
   - Example: 50% for affiliate, 10% for referee (total 60%)

3. **Validation**
   - Total must equal maximum commission rate
   - Cannot exceed maximum
   - Validates both rates

4. **Request Submission**
   - PUT request to `/v1/referral/rate`
   - Body:
     ```json
     {
       "affiliate_rate": 0.5,
       "referee_rate": 0.1
     }
     ```
   - Includes authentication headers

5. **Response Handling**
   - Success: Rate updated
   - Error: Invalid rate, etc.
   - Updates UI with new rates

## Network-Specific Behavior

### Mainnet Behavior

- **Base URL**: `https://api.orderly.org`
- **Real Commissions**: Earns real commissions from referrals
- **Real Rebates**: Receives real rebates on trading fees
- **Production Data**: Real trading volume and statistics
- **Real Applications**: Affiliate applications processed for real accounts

### Testnet Behavior

- **Base URL**: `https://testnet-api.orderly.org`
- **Test Commissions**: Test commissions (may not have real value)
- **Test Rebates**: Test rebates (may not have real value)
- **Test Data**: Testnet trading volume and statistics
- **Test Applications**: Test affiliate applications

### Network Detection

The affiliate components receive the network context through:

1. **OrderlyAppProvider Context**: Provides `networkId` to all child components
2. **API Client**: Automatically uses correct base URL based on `networkId`
3. **Broker ID**: Added to all API requests from `VITE_ORDERLY_BROKER_ID`

## Key Features

### Affiliate Dashboard

1. **Summary Statistics**
   - Total commission earned
   - 30-day commission
   - Number of referees that traded
   - Referral volume

2. **Referral Codes Management**
   - View all referral codes
   - Edit referral codes
   - Copy referral links
   - See remaining referral codes

3. **Referees List**
   - List of all referees
   - Referee addresses
   - Total commission from each referee
   - Total volume from each referee
   - Invitation time

4. **Commission History**
   - Historical commission earnings
   - Filter by time period
   - Commission breakdown by referee

5. **Commission Rate Settings**
   - Set maximum commission rate
   - Configure sharing between affiliate and referee
   - Adjust rates dynamically

### Trader Dashboard

1. **Rebate Statistics**
   - Total rebates earned
   - 30-day rebates
   - Trading volume with rebates
   - Current rebate rate

2. **Referrer Information**
   - Who referred them
   - Referral code used
   - Rebate rate received

3. **Rebate History**
   - Historical rebate earnings
   - Filter by time period
   - Rebate breakdown by trade

4. **Referral Code Binding**
   - Enter referral code
   - Bind code to account
   - View bound status

## Data Structures

### ReferralInfo Interface

```typescript
interface ReferralInfo {
  referral_code?: string;        // User's referral code (if affiliate)
  bound_referral_code?: string;  // Bound referral code (if trader)
  is_affiliate: boolean;         // Whether user is affiliate
  is_trader: boolean;            // Whether user is trader
  commission_rate?: number;      // Commission rate (if affiliate)
  rebate_rate?: number;          // Rebate rate (if trader)
  total_commission?: number;     // Total commission earned
  total_rebate?: number;         // Total rebate received
  referee_count?: number;        // Number of referees
}
```

### DailyVolume Interface

```typescript
interface DailyVolume {
  date: string;                  // Date
  volume: number;                // Trading volume
  commission?: number;           // Commission (if affiliate)
  rebate?: number;               // Rebate (if trader)
}
```

### UserVolumeType Interface

```typescript
interface UserVolumeType {
  "1d_volume"?: number;          // 1-day volume
  "7d_volume"?: number;          // 7-day volume
  "30d_volume"?: number;         // 30-day volume
  all_volume?: number;           // All-time volume
}
```

## Key Differences: Mainnet vs Testnet

| Aspect | Mainnet | Testnet |
|--------|---------|---------|
| **API Base URL** | `https://api.orderly.org` | `https://testnet-api.orderly.org` |
| **Commissions** | Real commissions | Test commissions |
| **Rebates** | Real rebates | Test rebates |
| **Trading Volume** | Real trading volume | Testnet trading volume |
| **Applications** | Production applications | Test applications |
| **Broker ID** | Production broker | Test broker |
| **Network ID** | `"mainnet"` | `"testnet"` |

## Important Notes

1. **Main Account Only**: Referral code generation and binding typically requires main account (not sub-accounts).

2. **Volume Requirements**: Auto-unlock of referral code requires $10,000 trading volume on main account (configurable).

3. **Commission Sharing**: Affiliates can share their commission with referees, creating a win-win situation.

4. **One Code Per User**: Users typically have one referral code, but can have multiple referees.

5. **Code Binding**: Traders can bind a referral code to start receiving rebates. Binding is usually one-time.

6. **Automatic Tracking**: Commissions and rebates are tracked automatically based on trading activity.

7. **Real-time Updates**: Statistics and earnings update in real-time as trades occur.

8. **Network Isolation**: Mainnet and testnet affiliate programs are completely separate.

9. **Broker-Specific**: Referral codes are broker-specific. Codes from one broker don't work on another.

10. **Application Process**: Users can apply for affiliate status to get higher commission rates than the auto-unlock (10%) rate.

## Example: Complete Referral Code Binding Flow

### Mainnet

```
1. User Opens Rewards Page
   - networkId = "mainnet"
   - API Base URL = "https://api.orderly.org"
   - User is on main account

2. User Clicks "Enter Code" (Trader Tab)
   - Opens referral code dialog
   - Shows input field

3. User Enters Code: "ABC123"
   - Validates format: ✓ (4-10 chars, uppercase, numbers)
   - Checks existence: Code exists ✓

4. User Clicks "Bind"
   - POST https://api.orderly.org/v1/referral/bind
   - Body: { "referral_code": "ABC123" }
   - Headers: Authentication (API key + signature)

5. Response Received
   - Success: Code bound successfully
   - UI updates to show bound status
   - User now receives rebates on trades

6. Future Trades
   - User trades normally
   - Rebates applied automatically
   - Rebates tracked and displayed in dashboard
```

## Example: Complete Referral Code Generation Flow

### Testnet

```
1. User Opens Rewards Page
   - networkId = "testnet"
   - API Base URL = "https://testnet-api.orderly.org"
   - User has traded $12,000 (eligible)

2. User Sees Affiliate Tab
   - Shows "Generate Code" button
   - Shows progress: $12,000 of $10,000 completed ✓

3. User Clicks "Generate Code"
   - POST https://testnet-api.orderly.org/v1/referral/generate
   - Body: { "auto_generate": true }
   - Headers: Authentication

4. Response Received
   - Success: Code generated (e.g., "XYZ789")
   - Code assigned to user
   - User can edit code if desired

5. User Edits Code (Optional)
   - Changes to "MYCODE123"
   - PUT https://testnet-api.orderly.org/v1/referral/code
   - Body: { "referral_code": "MYCODE123" }
   - Validates uniqueness ✓

6. User Shares Code
   - Gets referral link: https://dex.example.com?referral_code=MYCODE123
   - Shares with friends
   - Starts earning commissions when referees trade
```

## Example: Commission Rate Update Flow

```
1. User (Affiliate) Opens Settings
   - Current max commission: 60%
   - Current split: 50% affiliate, 10% referee

2. User Wants to Change Split
   - New split: 40% affiliate, 20% referee
   - Total still equals 60% ✓

3. User Submits
   - PUT https://api.orderly.org/v1/referral/rate
   - Body: {
       "affiliate_rate": 0.4,
       "referee_rate": 0.2
     }

4. Response Received
   - Success: Rate updated
   - New rates apply to future trades
   - Referees now receive 20% instead of 10%
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
2. Filter by "orderly" or "referral"
3. Look for:
   - `GET /v1/referral/info` - Get referral information
   - `POST /v1/referral/bind` - Bind referral code
   - `POST /v1/referral/generate` - Generate referral code
   - `PUT /v1/referral/code` - Update referral code
   - `PUT /v1/referral/rate` - Update commission rate
   - `GET /v1/referral/referees` - Get referees list
   - `GET /v1/referral/commission` - Get commission history

### Check Referral Status

The affiliate package provides hooks and context:
- `useReferralContext()` - Access referral context
- `ReferralContext` - React context for referral data
- Check `isAffiliate`, `isTrader`, `referralInfo` properties

## Summary

The rewards/affiliate system provides:

1. **Two Roles**: Affiliate (referrer) and Trader (referee)
2. **Referral Codes**: Generate, bind, and share referral codes
3. **Commissions**: Affiliates earn commissions from referrals
4. **Rebates**: Traders receive rebates on trading fees
5. **Network-Aware**: Uses correct API base URL based on `networkId`
6. **Broker-Specific**: Codes are broker-specific
7. **Real-time Tracking**: Automatic tracking of commissions and rebates
8. **Statistics**: Comprehensive dashboards for both roles

The system works identically on both mainnet and testnet, with the only difference being the API base URL and whether commissions/rebates have real value. Users can submit referral code bindings, generate codes, and update commission rates through the UI, with all operations authenticated and tracked automatically.

