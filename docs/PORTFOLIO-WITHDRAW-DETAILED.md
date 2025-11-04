# Portfolio Withdraw Function - Detailed Guide

This document provides a comprehensive explanation of the withdrawal functionality in the Portfolio Assets page, including cross-chain rebalancing, vault balance considerations, settlement requirements, and the complete withdrawal flow for both mainnet and testnet.

## Overview

The withdraw function allows users to transfer assets from their Orderly Network broker account to their Web3 wallet or another broker account. This process can involve:
- Direct withdrawals (same chain, sufficient vault balance)
- Cross-chain withdrawals (requires rebalancing)
- Withdrawals to other broker accounts (Account ID)
- Settlement of unsettled PnL before withdrawal

## Accessing Withdraw Function

**Route**: `/portfolio/assets`

**Entry Point**: 
- Click "Withdraw" button on the Assets page
- Opens withdrawal dialog/sheet component

## Withdrawal Flow Types

### 1. Direct Withdrawal (Bridgeless Networks - Sufficient Vault Balance)

**When**: 
- Withdrawing to a bridgeless network
- Vault balance on destination chain is sufficient
- No cross-chain rebalancing needed

**Process**:
1. User selects bridgeless network
2. User enters amount and recipient address
3. Settlement check (if unsettled PnL exists)
4. Direct withdrawal from vault to recipient
5. Funds arrive in recipient wallet

**Advantages**:
- Lower fees (no rebalancing fees)
- Faster processing
- Simpler transaction flow

### 2. Cross-Chain Withdrawal (Rebalancing Required)

**When**: 
- Withdrawing to a chain where vault balance is insufficient
- Requires cross-chain rebalancing to move funds to destination chain

**Process**:
1. User selects destination chain
2. User enters amount and recipient address
3. System checks vault balance on destination chain
4. If insufficient: Cross-chain rebalancing required
5. Rebalancing fee charged
6. Funds rebalanced to destination chain
7. Withdrawal processed
8. Funds arrive in recipient wallet

**Rebalancing Details**:
- Automatically triggered when vault balance < withdrawal amount
- Uses cross-chain bridge (likely Stargate or similar)
- Rebalancing fee applied
- Processing time: Longer than direct withdrawals

### 3. Withdrawal to Other Broker Account

**When**: Withdrawing to another Orderly Network broker account

**Process**:
1. User selects "Other {{brokerName}} account"
2. User enters Account ID (not wallet address)
3. System validates Account ID exists
4. Settlement check (if unsettled PnL exists)
5. Internal transfer to other broker account
6. Funds arrive in other account

**Key Differences**:
- Uses Account ID instead of wallet address
- No blockchain transaction needed
- Faster processing
- Different validation rules

## Complete Withdrawal Flow - Step by Step

### Step 1: Open Withdrawal Dialog

1. **User Navigates to Assets Page**
   - Route: `/portfolio/assets`
   - Network ID determined: `mainnet` or `testnet`
   - API Base URL set accordingly

2. **User Clicks "Withdraw" Button**
   - Opens withdrawal dialog/sheet
   - Shows withdrawal interface

### Step 2: Destination Network Selection

1. **Select Destination Chain**
   - User selects blockchain network (e.g., Ethereum, Arbitrum, BSC)
   - System validates:
     - Network supports withdrawals
     - Network is bridgeless or supports cross-chain
   - If unsupported: Shows error "Withdrawals are not supported on this chain. Please switch to any of the bridgeless networks."

2. **Token Selection**
   - Usually USDC (primary withdrawal token)
   - System shows available balance in broker account
   - Shows: "Available to withdraw"

3. **Available Balance Display**
   - Shows available balance in broker account
   - Excludes unsettled PnL
   - Updates based on account state

### Step 3: Amount Input

1. **User Enters Withdrawal Amount**
   - Input field for amount
   - "Max" button to use maximum available
   - Amount validation:
     - Must be positive number
     - Must be ≤ available balance
     - Must meet minimum withdrawal amount
     - Must account for fees (if any)

2. **System Validations**
   ```
   - Balance check: available_balance >= withdrawal_amount ✓
   - Minimum amount: withdrawal_amount >= min_withdrawal_amount ✓
   - Unsettled PnL check: unsettled_pnl = 0 (or must settle first) ✓
   ```

3. **Unsettled PnL Check**
   - If unsettled PnL exists:
     - Shows warning: "Unsettled balance can not be withdrawn. In order to withdraw, please settle your balance first."
     - Prompts user to settle first
     - Settlement required before withdrawal

### Step 4: Settlement (If Required)

**When Required**: If user has unsettled PnL

1. **Settlement Prompt**
   - System detects unsettled PnL
   - Shows "Settle" button or warning
   - Message: "Settlement will take up to 1 minute before you can withdraw your available balance."

2. **Settlement Process**
   - User clicks "Settle PnL"
   - Confirmation dialog shown
   - POST request to `/v1/client/settlement`
   - Settlement processing (up to 1 minute)
   - Unsettled PnL converted to available balance

3. **After Settlement**
   - Available balance updated
   - User can proceed with withdrawal

### Step 5: Recipient Selection

1. **Recipient Type Selection**
   - **Option 1: Wallet Address**
     - User enters wallet address
     - Validates address format (EVM address format)
     - Shows: "Recipient address"
   
   - **Option 2: Other Broker Account**
     - User selects "Other {{brokerName}} account"
     - User enters Account ID
     - Tip: "Please enter an Account ID instead of a wallet address."
     - Validates Account ID exists
     - If invalid: "Invalid Account ID. Please try again."

2. **Address/Account ID Validation**
   - Wallet address: Must be valid EVM address format
   - Account ID: Must exist in Orderly Network system
   - System validates before proceeding

### Step 6: Vault Balance Check

1. **Vault Balance Check**
   - System checks vault balance on destination chain
   - Compares with withdrawal amount
   - Determines if rebalancing needed

2. **Sufficient Vault Balance**
   - If vault_balance >= withdrawal_amount:
     - Direct withdrawal possible
     - No rebalancing fee
     - Faster processing

3. **Insufficient Vault Balance**
   - If vault_balance < withdrawal_amount:
     - Shows warning: "Withdrawal exceeds the balance of the {{networkName}} vault ( {{chainVaultBalance}} USDC ). Cross-chain rebalancing fee will be charged for withdrawal to {{networkName}}."
     - Cross-chain rebalancing required
     - Rebalancing fee will be charged
     - Longer processing time

4. **Vault Balance Warning**
   - Alternative message: "The balance of {{tokenName}} on the {{chainName}} is {{balance}}, which is insufficient to meet your withdrawal request. Please try again later or switch to another chain for withdrawal."

### Step 7: Cross-Chain Rebalancing (If Needed)

**When Required**: When vault balance on destination chain is insufficient

1. **Rebalancing Preview**
   - Shows rebalancing details:
     - Source: Vault on other chain
     - Destination: Vault on target chain
     - Amount: Amount needed for withdrawal
     - Rebalancing fee: Calculated fee
     - Estimated processing time

2. **Rebalancing Warning**
   - Warning: "Withdrawals that require cross-chain rebalancing can't be cancelled or followed up with more withdrawals until they've been processed."
   - User must wait for rebalancing to complete

3. **Rebalancing Process**
   - System initiates cross-chain rebalancing
   - Status: "Your cross-chain withdrawal is being processed..."
   - Funds moved from source vault to destination vault
   - Processing time: Varies (can be several minutes)

4. **Rebalancing Completion**
   - Vault balance updated on destination chain
   - Withdrawal can proceed

### Step 8: Withdrawal Request Submission

1. **Withdrawal Preview**
   - Shows final withdrawal details:
     - Withdrawal amount
     - Token: USDC (or selected token)
     - Recipient address/Account ID
     - Destination network
     - Fees (if any)
     - Estimated arrival time

2. **Confirmation Dialog**
   - For cross-chain: "Confirm to withdraw"
   - Shows all details for review
   - User must confirm before proceeding

3. **Withdrawal Request**
   - User clicks "Withdraw" or "Confirm to withdraw"
   - POST request to `/v1/asset/withdraw`
   - Request body:
     ```json
     {
       "amount": "1000",
       "token": "USDC",
       "chain_id": "42161",
       "address": "0xRecipientAddress..."
     }
     ```
   - OR for Account ID:
     ```json
     {
       "amount": "1000",
       "token": "USDC",
       "chain_id": "42161",
       "account_id": "account_id_123"
     }
     ```
   - Includes authentication headers (API key + signature)

4. **Request Validation**
   - System validates:
     - Amount is valid
     - Sufficient balance
     - Valid recipient (address or Account ID)
     - Network supports withdrawal
     - No pending withdrawals blocking

### Step 9: Withdrawal Processing

1. **Initial Status**
   - Status: "Withdraw requested"
   - Request queued for processing
   - System begins processing

2. **Processing Steps**
   - **Step 1: Rebalancing** (if needed)
     - Cross-chain rebalancing (if vault balance insufficient)
     - Status: "Rebalancing..."
   
   - **Step 2: Withdrawal Processing**
     - Withdrawal from vault to recipient
     - On-chain transaction (if to wallet address)
     - Internal transfer (if to Account ID)
     - Status: "Processing..."

3. **Transaction Monitoring**
   - System monitors transaction status
   - Updates status in real-time
   - Shows progress indicators

### Step 10: Withdrawal Completion

1. **Success Confirmation**
   - Status: "Withdraw completed"
   - Transaction hash displayed (if on-chain)
   - Link to block explorer provided (if on-chain)
   - Funds arrive at destination

2. **Account Update**
   - Broker account balance updated
   - Available balance decreased
   - Withdrawal history updated
   - UI refreshes to show new balance

3. **Completion**
   - Success message displayed
   - User can view withdrawal in history
   - Withdrawal dialog can be closed

## API Endpoints

### Submit Withdrawal Request

**Withdraw Assets**:
- **Mainnet**: `POST https://api.orderly.org/v1/asset/withdraw`
- **Testnet**: `POST https://testnet-api.orderly.org/v1/asset/withdraw`
- **Authentication**: Required (API key + signature)
- **Request Body**:
  ```json
  {
    "amount": "1000",
    "token": "USDC",
    "chain_id": "42161",
    "address": "0xRecipientAddress..."  // For wallet address
  }
  ```
  OR
  ```json
  {
    "amount": "1000",
    "token": "USDC",
    "chain_id": "42161",
    "account_id": "account_id_123"  // For other broker account
  }
  ```

### Get Withdrawal History

**Get Withdrawal History**:
- **Mainnet**: `GET https://api.orderly.org/v1/asset/withdraw`
- **Testnet**: `GET https://testnet-api.orderly.org/v1/asset/withdraw`
- **Authentication**: Required (API key + signature)
- **Query Parameters**:
  - `chain_id`: Filter by chain
  - `token`: Filter by token
  - `status`: Filter by status
  - `page`: Page number
  - `size`: Page size

**Response**:
```json
{
  "success": true,
  "data": {
    "rows": [
      {
        "id": "withdraw_id",
        "token": "USDC",
        "amount": "1000",
        "chain_id": "42161",
        "address": "0x...",
        "status": "completed",
        "tx_hash": "0x...",
        "created_time": 1234567890,
        "updated_time": 1234567890
      }
    ],
    "meta": {
      "total": 10,
      "page": 1,
      "size": 20
    }
  }
}
```

### Withdrawal Statuses

**Withdrawal Statuses**:
- `requested`: Withdrawal request submitted
- `pending`: Request pending processing
- `rebalancing`: Cross-chain rebalancing in progress
- `processing`: Withdrawal being processed
- `completed`: Successfully withdrawn
- `failed`: Withdrawal failed

## Network-Specific Behavior

### Mainnet Withdrawals

- **Real Assets**: Withdraws real tokens (USDC, etc.)
- **Real Transactions**: Actual on-chain transactions
- **Production Contracts**: 
  - Broker withdrawal contracts (mainnet)
  - Vault contracts (mainnet)
  - Cross-chain bridge (mainnet)
- **Real Gas Fees**: Actual transaction costs
- **Block Explorer**: Links to mainnet explorer (Etherscan, etc.)
- **Processing Time**: 
  - Direct: ~5-15 minutes
  - With rebalancing: ~15-30 minutes
  - To Account ID: ~1-5 minutes

### Testnet Withdrawals

- **Test Assets**: Withdraws testnet tokens
- **Test Transactions**: Testnet chain transactions
- **Test Contracts**: 
  - Broker withdrawal contracts (testnet)
  - Vault contracts (testnet)
  - Cross-chain bridge (testnet)
- **Free/Cheap Gas**: Testnet usually has free/cheap gas
- **Block Explorer**: Links to testnet explorer
- **Processing Time**: Similar to mainnet, but may be faster

## Fees and Costs

### 1. Rebalancing Fees

**Cross-Chain Rebalancing**:
- Charged when vault balance on destination chain is insufficient
- Fee varies by:
  - Source chain
  - Destination chain
  - Amount
  - Current network congestion
- Displayed in withdrawal preview
- Warning: "Cross-chain rebalancing fee will be charged"

### 2. Network Fees

**Transaction Fees**:
- Gas fees for on-chain transactions (if withdrawing to wallet)
- Paid by broker (not user) in most cases
- No fees for withdrawals to Account ID

### 3. Total Cost Calculation

```
Total Cost = Rebalancing Fees (if cross-chain rebalancing required)
```

**Note**: Users typically don't pay gas fees directly. The broker covers transaction costs, but rebalancing fees may be deducted from the withdrawal amount.

## Error Handling

### Common Errors

1. **Insufficient Balance**
   - Error: "Insufficient balance"
   - Cause: Available balance < withdrawal amount
   - Solution: Reduce withdrawal amount or settle PnL first

2. **Unsettled PnL**
   - Error: "Unsettled balance can not be withdrawn"
   - Cause: User has unsettled PnL that must be settled first
   - Solution: Settle PnL before withdrawing

3. **Minimum Amount Error**
   - Error: "quantity must large than {{minAmount}}"
   - Cause: Withdrawal amount < minimum withdrawal amount
   - Solution: Increase withdrawal amount

4. **Unsupported Chain**
   - Error: "Withdrawals are not supported on this chain. Please switch to any of the bridgeless networks."
   - Cause: Selected chain doesn't support withdrawals
   - Solution: Select a supported chain (bridgeless network)

5. **Invalid Recipient Address**
   - Error: Invalid address format
   - Cause: Invalid wallet address format
   - Solution: Enter valid EVM address

6. **Invalid Account ID**
   - Error: "Invalid Account ID. Please try again."
   - Cause: Account ID doesn't exist
   - Solution: Enter valid Account ID

7. **Insufficient Vault Balance**
   - Error: "The balance of {{tokenName}} on the {{chainName}} is {{balance}}, which is insufficient to meet your withdrawal request."
   - Cause: Vault balance insufficient, rebalancing in progress or unavailable
   - Solution: Try again later or select different chain

8. **Withdrawal Failed**
   - Error: "Withdraw failed"
   - Cause: Transaction failed or processing error
   - Solution: Check transaction status, retry if needed

9. **Pending Withdrawal**
   - Error: "There is a withdrawal in progress."
   - Cause: Previous withdrawal still processing
   - Solution: Wait for pending withdrawal to complete

10. **Cross-Chain Restriction**
    - Error: "Withdrawals that require cross-chain rebalancing can't be cancelled or followed up with more withdrawals until they've been processed."
    - Cause: Cross-chain rebalancing in progress
    - Solution: Wait for rebalancing to complete

## Important Notes

1. **Settlement Required**: Unsettled PnL must be settled before withdrawal. Settlement takes up to 1 minute.

2. **Vault Balance**: Vault balance on destination chain determines if rebalancing is needed. Rebalancing incurs fees.

3. **Bridgeless Networks**: Use bridgeless networks when possible to avoid rebalancing fees.

4. **Account ID vs Address**: Withdrawals to other broker accounts use Account ID, not wallet address.

5. **Cross-Chain Restrictions**: Cross-chain withdrawals can't be cancelled and block other withdrawals until processed.

6. **Minimum Amounts**: Each chain/token may have different minimum withdrawal amounts.

7. **Processing Time**: Withdrawals can take 5-30 minutes depending on rebalancing needs.

8. **Network Support**: Not all chains support withdrawals. Check supported networks before withdrawing.

9. **Pending Withdrawals**: Only one withdrawal can be processed at a time. Wait for pending withdrawals to complete.

10. **Mobile Device Limitation**: QR code linking from mobile devices doesn't enable withdrawals. Must disconnect and reconnect wallet directly.

## Example: Complete Withdrawal Flow

### Mainnet - Direct Withdrawal (USDC to Arbitrum)

```
1. User Opens Assets Page
   - networkId = "mainnet"
   - API Base URL = "https://api.orderly.org"
   - User has 5000 USDC in broker account
   - Available balance: 5000 USDC
   - Unsettled PnL: 0 USDC

2. User Clicks "Withdraw"
   - Opens withdrawal dialog
   - Selects: Arbitrum (chain ID: 42161)
   - Selects: USDC
   - Shows available balance: 5000 USDC

3. User Enters Amount: 1000 USDC
   - Validates: balance >= 1000 ✓
   - Validates: minimum amount ✓
   - Validates: unsettled PnL = 0 ✓

4. User Enters Recipient Address
   - Address: 0xRecipientAddress...
   - Validates address format ✓

5. Vault Balance Check
   - Checks vault balance on Arbitrum
   - Vault balance: 50000 USDC
   - Withdrawal amount: 1000 USDC
   - Sufficient ✓ (no rebalancing needed)

6. User Confirms Withdrawal
   - Reviews withdrawal details
   - Clicks "Withdraw"
   - POST https://api.orderly.org/v1/asset/withdraw
   - Body: {
       "amount": "1000",
       "token": "USDC",
       "chain_id": "42161",
       "address": "0xRecipientAddress..."
     }

7. Withdrawal Processing
   - Status: "Withdraw requested"
   - System processes withdrawal
   - On-chain transaction initiated
   - Status: "Processing..."

8. Withdrawal Completed
   - Status: "Withdraw completed"
   - Transaction hash: 0xabc123...
   - 1000 USDC sent to recipient address
   - Available balance: 4000 USDC
   - Withdrawal history updated
```

### Testnet - Cross-Chain Withdrawal (USDC to Ethereum Sepolia)

```
1. User Opens Assets Page
   - networkId = "testnet"
   - API Base URL = "https://testnet-api.orderly.org"
   - User has 2000 USDC in broker account
   - Available balance: 2000 USDC
   - Unsettled PnL: 0 USDC

2. User Clicks "Withdraw"
   - Opens withdrawal dialog
   - Selects: Ethereum Sepolia (chain ID: 11155111)
   - Selects: USDC
   - Shows available balance: 2000 USDC

3. User Enters Amount: 1500 USDC
   - Validates: balance >= 1500 ✓
   - Validates: minimum amount ✓
   - Validates: unsettled PnL = 0 ✓

4. User Enters Recipient Address
   - Address: 0xRecipientAddress...
   - Validates address format ✓

5. Vault Balance Check
   - Checks vault balance on Ethereum Sepolia
   - Vault balance: 500 USDC
   - Withdrawal amount: 1500 USDC
   - Insufficient ✗ (rebalancing needed)
   - Shows warning: "Withdrawal exceeds the balance of the Ethereum Sepolia vault (500 USDC). Cross-chain rebalancing fee will be charged."

6. User Confirms Withdrawal
   - Reviews withdrawal details including rebalancing fee
   - Clicks "Confirm to withdraw"
   - POST https://testnet-api.orderly.org/v1/asset/withdraw
   - Body: {
       "amount": "1500",
       "token": "USDC",
       "chain_id": "11155111",
       "address": "0xRecipientAddress..."
     }

7. Cross-Chain Rebalancing
   - Status: "Withdraw requested"
   - System initiates rebalancing
   - Status: "Your cross-chain withdrawal is being processed..."
   - Funds moved from other vault to Ethereum Sepolia vault
   - Rebalancing fee deducted
   - Processing time: ~10-15 minutes

8. Withdrawal Processing
   - Rebalancing completed
   - Vault balance updated: 2000 USDC
   - Withdrawal processing begins
   - Status: "Processing..."
   - On-chain transaction initiated

9. Withdrawal Completed
   - Status: "Withdraw completed"
   - Transaction hash: 0xdef456...
   - ~1495 USDC sent to recipient (1500 - rebalancing fee)
   - Available balance: 500 USDC
   - Withdrawal history updated
```

### Mainnet - Withdrawal to Other Broker Account

```
1. User Opens Assets Page
   - networkId = "mainnet"
   - API Base URL = "https://api.orderly.org"
   - User has 3000 USDC in broker account

2. User Clicks "Withdraw"
   - Opens withdrawal dialog
   - Selects: Arbitrum (chain ID: 42161)
   - Selects: USDC

3. User Selects "Other Broker Account"
   - Selects "Other {{brokerName}} account"
   - Enters Account ID: "account_id_123"
   - Validates Account ID exists ✓

4. User Enters Amount: 1000 USDC
   - Validates: balance >= 1000 ✓
   - Validates: unsettled PnL = 0 ✓

5. User Confirms Withdrawal
   - Reviews withdrawal details
   - Clicks "Withdraw"
   - POST https://api.orderly.org/v1/asset/withdraw
   - Body: {
       "amount": "1000",
       "token": "USDC",
       "chain_id": "42161",
       "account_id": "account_id_123"
     }

6. Withdrawal Processing
   - Status: "Withdraw requested"
   - Internal transfer to other account
   - Status: "Processing..."
   - Faster than on-chain withdrawal

7. Withdrawal Completed
   - Status: "Withdraw completed"
   - 1000 USDC transferred to account_id_123
   - Available balance: 2000 USDC
   - No transaction hash (internal transfer)
   - Withdrawal history updated
```

## Debugging

### Check Withdrawal Status

```javascript
// In browser console or via API
GET /v1/asset/withdraw?status=completed
```

### Check Transaction on Block Explorer

```javascript
// Get transaction hash from withdrawal history
// View on block explorer:
// Mainnet: https://etherscan.io/tx/{tx_hash}
// Testnet: https://sepolia.etherscan.io/tx/{tx_hash}
```

### Monitor Withdrawal Transactions

1. Open browser DevTools → Network tab
2. Filter by "orderly" or "asset"
3. Look for:
   - `POST /v1/asset/withdraw` - Submit withdrawal request
   - `GET /v1/asset/withdraw` - Fetch withdrawal history
   - Monitor WebSocket connections for real-time updates

### Check Vault Balance

```javascript
// Vault balance is checked automatically during withdrawal
// Check via API if available:
GET /v1/vault/balance?chain_id={chain_id}
```

### Check Account Balance

```javascript
// Check available balance:
GET /v1/client/balance
// Check for unsettled PnL:
GET /v1/client/info
```

## Summary

The withdraw function in the Portfolio Assets page provides a comprehensive way to withdraw assets from Orderly Network broker accounts to Web3 wallets or other broker accounts. It handles:

1. **Direct Withdrawals**: Simple withdrawals when vault balance is sufficient
2. **Cross-Chain Withdrawals**: With automatic rebalancing when vault balance is insufficient
3. **Account-to-Account Transfers**: Fast internal transfers to other broker accounts
4. **Settlement Requirements**: Automatic checks and prompts for unsettled PnL
5. **Vault Balance Management**: Automatic detection and rebalancing when needed

The process is fully automated and guides users through each step, with clear warnings about rebalancing fees and processing times. All withdrawals work identically on both mainnet and testnet, with the only difference being the networks, contracts, and whether assets have real value. The system ensures users can't withdraw unsettled PnL and provides clear feedback about vault balance and rebalancing requirements.

