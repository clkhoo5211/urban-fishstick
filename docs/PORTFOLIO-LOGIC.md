# Portfolio Logic - Mainnet & Testnet

This document explains how the portfolio functionality works in both mainnet and testnet environments, including asset management, deposits, withdrawals, transfers, and other portfolio operations.

## Overview

The portfolio functionality is implemented using the **Portfolio** package (`@orderly.network/portfolio` version 2.8.1), which provides a comprehensive portfolio management system. The portfolio includes multiple modules: Overview, Assets, Positions, Orders, History, Fee Tier, API Key Management, and Settings. Unlike other sections, the portfolio is primarily a **management and display interface** with some submission operations for asset transfers, settlements, and conversions.

## Architecture

### Component Structure

```
App.tsx
  └── OrderlyProvider (app/components/orderlyProvider/index.tsx)
      └── OrderlyAppProvider (from @orderly.network/react-app)
          └── PortfolioLayout (app/pages/portfolio/Layout.tsx)
              └── PortfolioLayoutWidget (from @orderly.network/portfolio)
                  ├── OverviewModule.OverviewPage (/portfolio)
                  ├── PositionsModule.PositionsPage (/portfolio/positions)
                  ├── OrdersModule.OrdersPage (/portfolio/orders)
                  ├── AssetsModule.AssetsPage (/portfolio/assets)
                  ├── HistoryModule.HistoryPage (/portfolio/history)
                  ├── FeeTierModule.FeeTierPage (/portfolio/feeTier)
                  ├── APIManagerModule.APIManagerPage (/portfolio/apiKey)
                  └── SettingModule.SettingPage (/portfolio/setting)
```

### Key Files

1. **Portfolio Layout**: `app/pages/portfolio/Layout.tsx`
   - Renders `PortfolioLayoutWidget` with sidebar navigation
   - Provides routing structure for portfolio pages

2. **Portfolio Pages**:
   - `Index.tsx`: Overview page
   - `Assets.tsx`: Assets management page
   - `Positions.tsx`: Positions page
   - `Orders.tsx`: Orders page
   - `History.tsx`: Trade history page
   - `Fee.tsx`: Fee tier page
   - `ApiKey.tsx`: API key management page
   - `Setting.tsx`: Settings page

3. **Portfolio Package**: `node_modules/@orderly.network/portfolio/`
   - Contains all portfolio logic, UI components, and API calls
   - Handles asset management, transfers, settlements, and data display

## Network Selection Logic

### How Network ID is Determined

The portfolio uses the same network selection logic as other Orderly components:

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
Portfolio components (from @orderly.network/portfolio) 
inherit network context from OrderlyAppProvider
    ↓
All portfolio API calls use the correct base URL
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

## Portfolio Modules

### 1. Overview Module

**Route**: `/portfolio`

**Purpose**: Display portfolio overview and summary

**Features**:
- Portfolio value
- Total assets
- Free collateral
- Unrealized PnL
- Unrealized ROI
- Current leverage
- Performance charts
- Asset distribution

**Data Display**:
- Real-time portfolio statistics
- Historical performance (7D, 30D, 90D)
- Asset breakdown
- PnL visualization

**No Submissions**: Display only

### 2. Assets Module

**Route**: `/portfolio/assets`

**Purpose**: Manage assets, deposits, withdrawals, and transfers

**Features**:
- View all assets and balances
- Deposit assets from Web3 wallet
- Withdraw assets to Web3 wallet
- Internal transfers (between accounts)
- Asset conversion (to USDC)
- Collateral management
- LTV (Loan-to-Value) monitoring

**Submissions**:
- Deposit
- Withdraw
- Internal Transfer
- Asset Conversion
- Settlement

### 3. Positions Module

**Route**: `/portfolio/positions`

**Purpose**: View and manage open positions

**Features**:
- List of open positions
- Position details (symbol, size, entry price, PnL)
- Close positions
- Navigate to trading page for symbol

**Submissions**:
- Close position (via trading page)

**No Direct Submissions**: Primarily display, positions managed via trading page

### 4. Orders Module

**Route**: `/portfolio/orders`

**Purpose**: View and manage orders

**Features**:
- List of open orders
- Order history
- Cancel orders
- Order details

**Submissions**:
- Cancel order

### 5. History Module

**Route**: `/portfolio/history`

**Purpose**: View trading history and transaction records

**Features**:
- Trade execution history
- Deposit/withdrawal history
- Transfer history
- Funding fee history
- Distribution history
- Filtering and pagination

**No Submissions**: Display only

### 6. Fee Tier Module

**Route**: `/portfolio/feeTier`

**Purpose**: Display fee tier information

**Features**:
- Current fee tier
- Fee tier requirements
- Fee structure (maker/taker fees)
- Volume requirements

**No Submissions**: Display only

### 7. API Key Module

**Route**: `/portfolio/apiKey`

**Purpose**: Manage API keys

**Features**:
- View API keys
- Create API keys
- Delete API keys
- Set API key permissions
- View API key usage

**Submissions**:
- Create API key
- Delete API key
- Update API key permissions

### 8. Setting Module

**Route**: `/portfolio/setting`

**Purpose**: Account and application settings

**Features**:
- Account settings
- Notification preferences
- Display preferences
- Security settings

**Submissions**:
- Update settings

## Portfolio API Endpoints

Based on the type definitions and typical Orderly Network API structure, the portfolio operations use the following endpoints:

### Mainnet Endpoints

#### Asset Management
- **Get Client Info**: `GET https://api.orderly.org/v1/client/info`
- **Get Holdings**: `GET https://api.orderly.org/v1/client/holding`
- **Get Balance**: `GET https://api.orderly.org/v1/client/balance`
- **Get Asset History**: `GET https://api.orderly.org/v1/asset/settlement_history`
- **Get Daily PnL**: `GET https://api.orderly.org/v1/public/daily_pnl`

#### Deposit/Withdraw
- **Deposit**: `POST https://api.orderly.org/v1/asset/deposit` (via on-chain transaction)
- **Withdraw**: `POST https://api.orderly.org/v1/asset/withdraw` (via on-chain transaction)
- **Get Deposit History**: `GET https://api.orderly.org/v1/asset/deposit`
- **Get Withdrawal History**: `GET https://api.orderly.org/v1/asset/withdraw`

#### Internal Transfer
- **Internal Transfer**: `POST https://api.orderly.org/v1/asset/internal_transfer`
- **Get Transfer History**: `GET https://api.orderly.org/v1/asset/internal_transfer`

#### Settlement
- **Settle PnL**: `POST https://api.orderly.org/v1/client/settlement`
- **Get Settlement History**: `GET https://api.orderly.org/v1/asset/settlement_history`

#### Positions & Orders
- **Get Positions**: `GET https://api.orderly.org/v1/positions`
- **Get Orders**: `GET https://api.orderly.org/v1/orders`
- **Cancel Order**: `DELETE https://api.orderly.org/v1/order/{order_id}`
- **Get Executions**: `GET https://api.orderly.org/v1/execution`

#### API Key Management
- **Get API Keys**: `GET https://api.orderly.org/v1/client/api_key`
- **Create API Key**: `POST https://api.orderly.org/v1/client/api_key`
- **Delete API Key**: `DELETE https://api.orderly.org/v1/client/api_key/{key_id}`

### Testnet Endpoints

All endpoints use the testnet base URL: `https://testnet-api.orderly.org`

**Note**: The exact endpoint paths may vary. These are inferred from the API structure and typical portfolio patterns.

## Submission Operations

### 1. Deposit Assets

#### Step-by-Step Process

1. **User Opens Assets Page**
   - Navigates to `/portfolio/assets`
   - Clicks "Deposit" button
   - Opens deposit dialog/sheet

2. **User Selects Network and Token**
   - Selects source chain (e.g., Ethereum, Arbitrum)
   - Selects token (e.g., USDC, ETH)
   - System shows available balance on selected chain

3. **User Enters Amount**
   - Enters deposit amount
   - System validates:
     - Sufficient balance on source chain
     - Minimum deposit amount
     - Gas fees availability

4. **Transaction Preparation**
   - If cross-chain: Bridge transaction required
   - If same chain: Direct deposit
   - System calculates:
     - Gas fees
     - Bridge fees (if applicable)
     - Estimated arrival time

5. **Token Approval** (if needed)
   - For ERC-20 tokens, approval required
   - User signs approval transaction
   - Waits for confirmation

6. **Deposit Transaction**
   - User reviews deposit details
   - Clicks "Deposit"
   - Wallet prompts for transaction signature
   - User signs transaction
   - Transaction submitted to blockchain

7. **Transaction Confirmation**
   - System monitors transaction status
   - Shows pending state
   - Waits for blockchain confirmation
   - Updates UI on success/failure

8. **Asset Arrival**
   - Assets arrive in broker account
   - Available balance updates
   - Deposit history updated

### 2. Withdraw Assets

#### Step-by-Step Process

1. **User Opens Assets Page**
   - Navigates to `/portfolio/assets`
   - Clicks "Withdraw" button
   - Opens withdrawal dialog/sheet

2. **User Selects Network and Token**
   - Selects destination chain
   - Selects token (usually USDC)
   - System shows available balance

3. **User Enters Amount**
   - Enters withdrawal amount
   - System validates:
     - Sufficient balance in broker account
     - Minimum withdrawal amount
     - Unsettled PnL (may need settlement first)

4. **Recipient Address**
   - User enters recipient wallet address
   - OR selects "Other Broker Account" (Account ID)
   - System validates address format

5. **Cross-Chain Considerations**
   - If cross-chain withdrawal:
     - Shows cross-chain rebalancing fee
     - Warns about vault balance
     - Shows estimated arrival time

6. **Settlement Check**
   - If unsettled PnL exists:
     - Prompts user to settle first
     - Settlement required before withdrawal

7. **Withdrawal Request**
   - User reviews withdrawal details
   - Clicks "Withdraw"
   - POST request to `/v1/asset/withdraw`
   - Body:
     ```json
     {
       "amount": "1000",
       "token": "USDC",
       "chain_id": "42161",
       "address": "0x..."
     }
     ```
   - Includes authentication headers

8. **Withdrawal Processing**
   - Request submitted
   - Status: "Withdraw requested"
   - System processes withdrawal
   - Funds transferred to destination

9. **Completion**
   - Status: "Withdraw completed"
   - Funds arrive at destination
   - Withdrawal history updated

### 3. Internal Transfer

#### Step-by-Step Process

1. **User Opens Assets Page**
   - Navigates to `/portfolio/assets`
   - Clicks "Transfer" button
   - Opens transfer dialog

2. **User Selects Accounts**
   - Selects "From" account (main or sub-account)
   - Selects "To" account (Account ID)
   - System validates account IDs

3. **User Enters Amount**
   - Enters transfer amount
   - System validates:
     - Sufficient balance
     - Unsettled PnL (must settle first)
     - Account type restrictions

4. **Settlement Check**
   - If unsettled balance exists:
     - Must settle PnL first
     - Settlement takes up to 1 minute

5. **Transfer Request**
   - User reviews transfer details
   - Clicks "Transfer"
   - POST request to `/v1/asset/internal_transfer`
   - Body:
     ```json
     {
       "from_account": "account_id_1",
       "to_account": "account_id_2",
       "amount": "1000",
       "token": "USDC"
     }
     ```
   - Includes authentication headers

6. **Transfer Processing**
   - Request submitted
   - Funds transferred internally
   - Available in 15 seconds

7. **Completion**
   - Transfer completed
   - Balance updated in both accounts
   - Transfer history updated

### 4. Settlement (Settle PnL)

#### Step-by-Step Process

1. **User Has Unsettled PnL**
   - System detects unsettled PnL
   - Shows "Settle" button or warning

2. **User Initiates Settlement**
   - Clicks "Settle PnL" button
   - Confirmation dialog shown
   - Warning: "Settlement will take up to 1 minute"

3. **Settlement Request**
   - User confirms settlement
   - POST request to `/v1/client/settlement`
   - Body: (empty or minimal)
   - Includes authentication headers

4. **Settlement Processing**
   - Request submitted
   - Status: "Settlement requested"
   - System settles PnL
   - Takes up to 1 minute

5. **Completion**
   - Status: "Settlement completed"
   - Unsettled PnL converted to settled balance
   - User can now withdraw/transfer

**Note**: Settlement is only allowed once every 10 minutes.

### 5. Asset Conversion (Convert to USDC)

#### Step-by-Step Process

1. **User Has Non-USDC Collateral**
   - User holds collateral in other tokens (e.g., ETH, BTC)
   - System shows conversion option

2. **User Initiates Conversion**
   - Clicks "Convert" button
   - Selects token to convert
   - Enters amount

3. **Conversion Preview**
   - System shows:
     - Convert rate
     - Amount to receive (USDC)
     - Conversion fee
     - Minimum received

4. **LTV Check**
   - System checks LTV (Loan-to-Value)
   - If LTV exceeds threshold:
     - Automatic conversion may occur
     - Manual conversion recommended

5. **Conversion Request**
   - User reviews conversion details
   - Clicks "Convert"
   - POST request to `/v1/client/convert` (inferred)
   - Body:
     ```json
     {
       "from_token": "ETH",
       "to_token": "USDC",
       "amount": "1.5"
     }
     ```
   - Includes authentication headers

6. **Conversion Processing**
   - Request submitted
   - System converts assets
   - Updates balances

7. **Completion**
   - Conversion completed
   - USDC balance updated
   - Original token balance decreased

### 6. Cancel Order

#### Step-by-Step Process

1. **User Views Orders**
   - Navigates to `/portfolio/orders`
   - Views list of open orders

2. **User Selects Order**
   - Clicks on order to cancel
   - OR clicks "Cancel" button

3. **Cancellation Request**
   - User confirms cancellation
   - DELETE request to `/v1/order/{order_id}`
   - Includes authentication headers

4. **Cancellation Processing**
   - Request submitted
   - Order cancelled
   - UI updates

5. **Completion**
   - Order removed from open orders
   - Order history updated

### 7. Create/Delete API Key

#### Create API Key

1. **User Opens API Key Page**
   - Navigates to `/portfolio/apiKey`
   - Clicks "Create API Key"

2. **User Configures Key**
   - Sets permissions (read, trade, etc.)
   - Sets IP restrictions (optional)
   - Sets name/description

3. **Key Generation**
   - POST request to `/v1/client/api_key`
   - Body:
     ```json
     {
       "name": "My API Key",
       "permissions": ["read", "trade"],
       "ip_whitelist": ["1.2.3.4"]
     }
     ```
   - Includes authentication headers

4. **Key Display**
   - API key and secret displayed
   - User must save secret (shown only once)
   - Key added to list

#### Delete API Key

1. **User Selects Key**
   - Views list of API keys
   - Selects key to delete

2. **Deletion Request**
   - User confirms deletion
   - DELETE request to `/v1/client/api_key/{key_id}`
   - Includes authentication headers

3. **Deletion Processing**
   - Key deleted
   - UI updates

## Network-Specific Behavior

### Mainnet Behavior

- **Base URL**: `https://api.orderly.org`
- **Real Assets**: Real tokens and balances
- **Real Transactions**: Actual on-chain transactions
- **Real Gas Fees**: Real ETH/gas costs
- **Production Contracts**: Production deposit/withdraw contracts
- **Real Settlements**: Actual PnL settlements

### Testnet Behavior

- **Base URL**: `https://testnet-api.orderly.org`
- **Test Assets**: Testnet tokens and balances
- **Test Transactions**: Testnet chain transactions
- **Free/Cheap Gas**: Testnet usually has free/cheap gas
- **Test Contracts**: Testnet deposit/withdraw contracts
- **Test Settlements**: Test PnL settlements

### Network Detection

The portfolio components receive the network context through:

1. **OrderlyAppProvider Context**: Provides `networkId` to all child components
2. **API Client**: Automatically uses correct base URL based on `networkId`
3. **Authentication**: All authenticated endpoints use API key + signature

## Key Features

### Asset Management

1. **Multi-Chain Support**
   - Deposit from multiple chains
   - Withdraw to multiple chains
   - Cross-chain bridging
   - Chain-specific balances

2. **Collateral Management**
   - Multiple collateral types
   - LTV monitoring
   - Automatic conversion
   - Collateral ratio tracking

3. **Balance Types**
   - Available balance
   - Frozen balance
   - Unsettled PnL
   - Pending short

4. **Account Types**
   - Main account
   - Sub-accounts
   - Internal transfers between accounts

### Transfer Types

1. **Deposit**: From Web3 wallet to broker account
2. **Withdraw**: From broker account to Web3 wallet
3. **Internal Transfer**: Between broker accounts (Account ID to Account ID)
4. **Cross-Chain**: Requires bridging and vault rebalancing

### Settlement System

1. **Unsettled PnL**: PnL from closed positions not yet settled
2. **Settlement**: Convert unsettled PnL to available balance
3. **Settlement Restrictions**: Once every 10 minutes
4. **Settlement Time**: Up to 1 minute processing time

## Key Differences: Mainnet vs Testnet

| Aspect | Mainnet | Testnet |
|--------|---------|---------|
| **API Base URL** | `https://api.orderly.org` | `https://testnet-api.orderly.org` |
| **Assets** | Real tokens | Testnet tokens |
| **Transactions** | Real on-chain | Testnet on-chain |
| **Gas Fees** | Real costs | Free/cheap |
| **Contracts** | Production contracts | Testnet contracts |
| **Settlements** | Real PnL | Test PnL |
| **Balances** | Real balances | Test balances |

## Important Notes

1. **Authentication Required**: All submission operations require API key authentication.

2. **Settlement Required**: Unsettled PnL must be settled before withdrawal/transfer.

3. **Settlement Cooldown**: Settlement only allowed once every 10 minutes.

4. **Internal Transfer Speed**: Internal transfers complete in 15 seconds.

5. **Cross-Chain Fees**: Cross-chain deposits/withdrawals incur bridge and rebalancing fees.

6. **Bridgeless Networks**: Some networks support direct deposits/withdrawals without bridging.

7. **Account ID vs Wallet Address**: Internal transfers use Account ID, not wallet address.

8. **Minimum Amounts**: Deposits and withdrawals have minimum amount requirements.

9. **Vault Balance**: Withdrawals may require cross-chain rebalancing if vault balance is insufficient.

10. **Network Isolation**: Mainnet and testnet portfolios are completely separate.

## Example: Complete Deposit Flow

### Mainnet Deposit

```
1. User Opens Assets Page
   - networkId = "mainnet"
   - API Base URL = "https://api.orderly.org"
   - User has 5000 USDC on Arbitrum

2. User Clicks "Deposit"
   - Opens deposit dialog
   - Selects: Arbitrum (chain ID: 42161)
   - Selects: USDC
   - Shows balance: 5000 USDC

3. User Enters Amount: 1000 USDC
   - Validates: balance >= 1000 ✓
   - Validates: minimum amount ✓
   - Shows gas fee estimate

4. User Approves (if needed)
   - Checks USDC allowance
   - If insufficient, prompts approval
   - User signs approval transaction
   - Waits for confirmation

5. User Confirms Deposit
   - Reviews deposit details
   - Clicks "Deposit"
   - Wallet prompts for signature
   - User signs transaction

6. Transaction Submitted
   - Transaction hash: 0xabc...
   - Status: "Deposit requested"
   - System monitors transaction

7. Transaction Confirmed
   - Block confirmation received
   - Status: "Deposit completed"
   - 1000 USDC added to broker account
   - Available balance updated
```

## Example: Complete Withdrawal Flow

### Testnet Withdrawal

```
1. User Opens Assets Page
   - networkId = "testnet"
   - API Base URL = "https://testnet-api.orderly.org"
   - User has 2000 USDC in broker account

2. User Clicks "Withdraw"
   - Opens withdrawal dialog
   - Selects: Arbitrum Sepolia (chain ID: 421614)
   - Selects: USDC
   - Shows available balance: 2000 USDC

3. User Enters Amount: 500 USDC
   - Validates: balance >= 500 ✓
   - Validates: minimum amount ✓
   - Checks unsettled PnL: 0 ✓

4. User Enters Recipient
   - Address: 0xRecipient...
   - Validates address format ✓

5. User Confirms Withdrawal
   - Reviews withdrawal details
   - Clicks "Withdraw"
   - POST https://testnet-api.orderly.org/v1/asset/withdraw
   - Body: {
       "amount": "500",
       "token": "USDC",
       "chain_id": "421614",
       "address": "0xRecipient..."
     }

6. Withdrawal Processing
   - Request submitted
   - Status: "Withdraw requested"
   - System processes withdrawal

7. Completion
   - Status: "Withdraw completed"
   - 500 USDC transferred to recipient
   - Balance updated: 1500 USDC remaining
```

## Example: Internal Transfer Flow

```
1. User Opens Assets Page
   - User has main account and sub-account
   - Main account balance: 10000 USDC
   - Sub-account balance: 0 USDC

2. User Clicks "Transfer"
   - Opens transfer dialog
   - From: Main account (account_id_1)
   - To: Sub-account (account_id_2)
   - Amount: 2000 USDC

3. User Enters Details
   - Validates: balance >= 2000 ✓
   - Validates: unsettled PnL = 0 ✓
   - Validates: account IDs exist ✓

4. User Confirms Transfer
   - Reviews transfer details
   - Clicks "Transfer"
   - POST https://api.orderly.org/v1/asset/internal_transfer
   - Body: {
       "from_account": "account_id_1",
       "to_account": "account_id_2",
       "amount": "2000",
       "token": "USDC"
     }

5. Transfer Processing
   - Request submitted
   - Funds transferred internally
   - Available in 15 seconds

6. Completion
   - Transfer completed
   - Main account: 8000 USDC
   - Sub-account: 2000 USDC
   - Transfer history updated
```

## Example: Settlement Flow

```
1. User Has Unsettled PnL
   - User closed a position with profit
   - Unsettled PnL: +500 USDC
   - Available balance: 1000 USDC
   - Cannot withdraw 1500 USDC (unsettled)

2. User Clicks "Settle PnL"
   - Settlement dialog shown
   - Warning: "Settlement will take up to 1 minute"

3. User Confirms Settlement
   - POST https://api.orderly.org/v1/client/settlement
   - Body: {}
   - Headers: Authentication

4. Settlement Processing
   - Request submitted
   - Status: "Settlement requested"
   - System settles PnL
   - Takes ~1 minute

5. Completion
   - Status: "Settlement completed"
   - Unsettled PnL: 0 USDC
   - Available balance: 1500 USDC
   - User can now withdraw/transfer
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
2. Filter by "orderly" or "asset"
3. Look for:
   - `GET /v1/client/info` - Get client information
   - `GET /v1/client/holding` - Get holdings
   - `POST /v1/asset/deposit` - Deposit request
   - `POST /v1/asset/withdraw` - Withdrawal request
   - `POST /v1/asset/internal_transfer` - Internal transfer
   - `POST /v1/client/settlement` - Settlement request
   - `GET /v1/positions` - Get positions
   - `GET /v1/orders` - Get orders
   - `DELETE /v1/order/{id}` - Cancel order

## Summary

The portfolio functionality provides:

1. **Asset Management**: Deposit, withdraw, transfer, and convert assets
2. **Position Management**: View and manage open positions
3. **Order Management**: View and cancel orders
4. **History Tracking**: View trade history and transactions
5. **Settlement**: Settle PnL to make funds available
6. **API Key Management**: Create and manage API keys
7. **Settings**: Configure account and application settings
8. **Network-Aware**: Uses correct API base URL based on `networkId`

The portfolio is primarily a **management interface** with some submission operations. Most operations require authentication and work identically on both mainnet and testnet, with the only difference being the API base URL and whether assets have real value. Users can submit deposits, withdrawals, transfers, settlements, and API key operations through authenticated API calls.

