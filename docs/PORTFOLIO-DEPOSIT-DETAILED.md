# Portfolio Deposit Function - Detailed Guide

This document provides a comprehensive explanation of the deposit functionality in the Portfolio Assets page, including cross-chain bridging, token approvals, and the complete deposit flow for both mainnet and testnet.

## Overview

The deposit function allows users to transfer assets from their Web3 wallet to their Orderly Network broker account. This process can involve:
- Direct deposits (same chain)
- Cross-chain bridging (via Stargate)
- Token swaps (if depositing non-USDC tokens)
- Token approvals (for ERC-20 tokens)

## Accessing Deposit Function

**Route**: `/portfolio/assets`

**Entry Point**: 
- Click "Deposit" button on the Assets page
- Opens deposit dialog/sheet component

## Deposit Flow Types

### 1. Direct Deposit (Bridgeless Networks)

**When**: Depositing on a chain that supports direct deposits without bridging

**Supported Chains**: 
- Bridgeless networks (varies by broker configuration)
- Check `VITE_ORDERLY_MAINNET_CHAINS` or `VITE_ORDERLY_TESTNET_CHAINS` for bridgeless networks

**Process**:
1. User selects bridgeless network
2. User selects token (usually USDC)
3. User enters amount
4. Token approval (if needed)
5. Direct deposit transaction to broker deposit contract
6. Assets arrive in broker account

**Advantages**:
- Lower fees (no bridge fees)
- Faster processing
- Simpler transaction flow

### 2. Cross-Chain Deposit (Bridging Required)

**When**: Depositing from a chain that requires bridging to reach the broker's network

**Process**:
1. User selects source chain
2. User selects token (e.g., USDC, ETH)
3. User enters amount
4. **Bridge Transaction** (via Stargate)
   - Assets bridged from source chain to destination chain
   - Bridge fees charged
5. **Deposit Transaction** (on destination chain)
   - Assets deposited to broker account
6. Assets arrive in broker account

**Bridging Details**:
- Uses Stargate bridge protocol
- Example: "Bridge to Arbitrum via Stargate"
- Cross-chain fees apply
- Longer processing time

### 3. Swap + Deposit (Token Conversion)

**When**: Depositing non-USDC tokens that need to be converted

**Process**:
1. User selects source chain
2. User selects token (e.g., ETH, BTC)
3. User enters amount
4. **Swap Transaction** (if needed)
   - Token swapped to USDC (via WOOFi)
   - Swap fee: 0.025% charged by WOOFi
   - Slippage tolerance configurable
5. **Bridge Transaction** (if cross-chain)
   - USDC bridged to destination chain
6. **Deposit Transaction**
   - USDC deposited to broker account
7. Assets arrive as USDC in broker account

**Swap Details**:
- WOOFi swap service used
- 0.025% swap fee
- Slippage tolerance must be set
- Minimum received amount calculated

## Complete Deposit Flow - Step by Step

### Step 1: Open Deposit Dialog

1. **User Navigates to Assets Page**
   - Route: `/portfolio/assets`
   - Network ID determined: `mainnet` or `testnet`
   - API Base URL set accordingly

2. **User Clicks "Deposit" Button**
   - Opens deposit dialog/sheet
   - Shows deposit interface

### Step 2: Network and Token Selection

1. **Select Source Chain**
   - User selects blockchain network (e.g., Ethereum, Arbitrum, BSC)
   - System shows:
     - Available tokens on selected chain
     - User's wallet balance on that chain
     - Network fees estimation

2. **Select Token**
   - User selects token to deposit (e.g., USDC, ETH, BTC)
   - System shows:
     - Available balance in wallet
     - Token symbol and name
     - Whether swap is needed (if not USDC)

3. **Balance Display**
   - Shows: "Your Web3 Wallet" balance
   - Shows: "Your {{brokerName}} account" balance (destination)
   - Updates based on selected network

### Step 3: Amount Input

1. **User Enters Deposit Amount**
   - Input field for amount
   - "Max" button to use maximum available
   - Amount validation:
     - Must be positive number
     - Must be ≤ available balance
     - Must meet minimum deposit amount
     - Must leave enough for gas fees

2. **System Validations**
   ```
   - Balance check: wallet_balance >= deposit_amount + gas_fees ✓
   - Minimum amount: deposit_amount >= min_deposit_amount ✓
   - Gas fee check: sufficient native token for gas ✓
   ```

3. **Preview Information**
   - Shows estimated fees:
     - Gas fees (source chain)
     - Bridge fees (if cross-chain)
     - Swap fees (if swapping)
     - Destination gas fees (if needed)
   - Shows estimated arrival time
   - Shows minimum received (if swapping)

### Step 4: Token Approval (ERC-20 Tokens)

**When Required**: For ERC-20 tokens (USDC, etc.) that need approval before transfer

1. **Check Allowance**
   - System checks current token allowance
   - Compares with deposit amount
   - If insufficient: approval required

2. **Approval Transaction**
   - User clicks "Approve" button
   - Wallet prompts for approval transaction
   - Transaction details:
     - Token: Selected token address
     - Spender: Broker deposit contract address
     - Amount: Deposit amount (or max)
   - User signs approval transaction
   - Transaction submitted to blockchain

3. **Approval Confirmation**
   - System monitors approval transaction
   - Waits for blockchain confirmation
   - Updates UI: "Approve success" or "Approve failed"
   - Proceeds to deposit if successful

### Step 5: Swap Transaction (If Needed)

**When Required**: Depositing non-USDC tokens

1. **Swap Preview**
   - Shows swap details:
     - From: Selected token (e.g., ETH)
     - To: USDC
     - Exchange rate
     - Swap fee: 0.025%
     - Minimum received amount
     - Slippage tolerance
   - Warning: "Please note that swap fees will be charged."

2. **Slippage Configuration**
   - Default slippage tolerance set
   - User can adjust (if configurable)
   - Warning: "Your transaction will revert if the price changes unfavorably by more than this percentage."

3. **Swap Transaction**
   - User clicks "Confirm to swap"
   - Wallet prompts for swap transaction
   - User signs transaction
   - Transaction submitted to WOOFi swap contract
   - System monitors transaction status

4. **Swap Confirmation**
   - Swap completed
   - User receives USDC on source chain
   - Proceeds to bridge/deposit

### Step 6: Bridge Transaction (If Cross-Chain)

**When Required**: Depositing from a chain that requires bridging

1. **Bridge Preview**
   - Shows bridge details:
     - Source chain: Selected chain
     - Destination chain: Broker's network (e.g., Arbitrum)
     - Bridge protocol: Stargate
     - Bridge fee: Calculated by Stargate
     - Estimated bridge time
   - Warning: "Cross-chain transaction fees will be charged. To avoid these, use our supported Bridgeless networks"

2. **Bridge Transaction**
   - User clicks "Bridge" button
   - Status: "Bridging"
   - Wallet prompts for bridge transaction
   - Transaction details:
     - Source chain
     - Destination chain
     - Amount
     - Bridge fee
   - User signs transaction
   - Transaction submitted to Stargate bridge contract

3. **Bridge Processing**
   - System monitors bridge transaction
   - Status: "Bridging..."
   - Waits for cross-chain confirmation
   - Assets arrive on destination chain

4. **Bridge Confirmation**
   - Bridge completed
   - Assets now on destination chain
   - Proceeds to deposit

### Step 7: Deposit Transaction

1. **Deposit Preview**
   - Shows final deposit details:
     - Amount: Final deposit amount (after swap/bridge if applicable)
     - Token: USDC (or selected token)
     - Destination: Broker account
     - Network: Destination chain
   - Shows: "Deposit to {{brokerName}}"
   - Estimated arrival time

2. **Destination Gas Fee Check**
   - If destination chain requires gas:
     - Shows: "Destination gas fee"
     - Description: "Additional gas tokens are required to cover operations on the destination chain."
     - User must have sufficient native token on destination chain

3. **Deposit Transaction**
   - User clicks "Deposit" button
   - Status: "Depositing"
   - Wallet prompts for deposit transaction
   - Transaction details:
     - To: Broker deposit contract address
     - Amount: Deposit amount
     - Token: USDC (or selected token)
     - Chain: Destination chain
   - User signs transaction
   - Transaction submitted to blockchain

4. **Transaction Monitoring**
   - System monitors deposit transaction
   - Status: "Deposit requested"
   - Shows transaction hash
   - Waits for blockchain confirmation

### Step 8: Transaction Confirmation

1. **Blockchain Confirmation**
   - Transaction confirmed on blockchain
   - Status: "Deposit completed"
   - Transaction hash displayed
   - Link to block explorer provided

2. **Broker Account Update**
   - Assets arrive in broker account
   - Available balance updates
   - Deposit history updated
   - UI refreshes to show new balance

3. **Completion**
   - Success message displayed
   - User can view deposit in history
   - Deposit dialog can be closed

## API Endpoints

### Deposit History

**Get Deposit History**:
- **Mainnet**: `GET https://api.orderly.org/v1/asset/deposit`
- **Testnet**: `GET https://testnet-api.orderly.org/v1/asset/deposit`
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
        "id": "deposit_id",
        "token": "USDC",
        "amount": "1000",
        "chain_id": "42161",
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

### Deposit Status

**Deposit Statuses**:
- `requested`: Deposit transaction submitted
- `pending`: Transaction pending confirmation
- `processing`: Being processed by broker
- `completed`: Successfully deposited
- `failed`: Deposit failed

## Network-Specific Behavior

### Mainnet Deposits

- **Real Assets**: Deposits real tokens (USDC, ETH, etc.)
- **Real Transactions**: Actual on-chain transactions
- **Production Contracts**: 
  - Broker deposit contracts (mainnet)
  - Stargate bridge (mainnet)
  - WOOFi swap (mainnet)
- **Real Gas Fees**: Actual ETH/gas costs
- **Block Explorer**: Links to mainnet explorer (Etherscan, etc.)
- **Processing Time**: 
  - Direct: ~1-5 minutes
  - Cross-chain: ~5-15 minutes
  - With swap: ~5-20 minutes

### Testnet Deposits

- **Test Assets**: Deposits testnet tokens
- **Test Transactions**: Testnet chain transactions
- **Test Contracts**: 
  - Broker deposit contracts (testnet)
  - Stargate bridge (testnet)
  - WOOFi swap (testnet)
- **Free/Cheap Gas**: Testnet usually has free/cheap gas
- **Block Explorer**: Links to testnet explorer
- **Processing Time**: Similar to mainnet, but may be faster

## Fees and Costs

### 1. Gas Fees

**Source Chain Gas**:
- Required for approval transaction (if needed)
- Required for swap transaction (if needed)
- Required for bridge transaction (if cross-chain)
- Required for deposit transaction
- Paid in native token (ETH, BNB, etc.)

**Destination Chain Gas**:
- Required for deposit transaction on destination chain
- Additional gas tokens may be needed
- Shows as "Destination gas fee"

### 2. Bridge Fees

**Stargate Bridge**:
- Charged when bridging assets cross-chain
- Fee varies by:
  - Source chain
  - Destination chain
  - Amount
  - Current network congestion
- Displayed in bridge preview

### 3. Swap Fees

**WOOFi Swap**:
- 0.025% fee on swap amount
- Charged when converting tokens to USDC
- Example: 1000 USDC swap = 0.25 USDC fee
- Displayed in swap preview

### 4. Total Cost Calculation

```
Total Cost = Gas Fees + Bridge Fees (if applicable) + Swap Fees (if applicable)
```

## Error Handling

### Common Errors

1. **Insufficient Balance**
   - Error: "Insufficient balance"
   - Cause: Wallet balance < deposit amount + fees
   - Solution: Reduce deposit amount or add funds

2. **Insufficient Allowance**
   - Error: "Insufficient allowance"
   - Cause: Token approval amount < deposit amount
   - Solution: Approve more tokens or approve max

3. **Rejected Transaction**
   - Error: "Rejected transaction"
   - Cause: User rejected transaction in wallet
   - Solution: Retry and approve transaction

4. **Minimum Amount Error**
   - Error: "quantity must large than {{minAmount}}"
   - Cause: Deposit amount < minimum deposit amount
   - Solution: Increase deposit amount

5. **Gas Fee Error**
   - Error: "Please ensure you have enough {{token}} for gas fees."
   - Cause: Insufficient native token for gas
   - Solution: Add native token to wallet

6. **Collateral Cap Error**
   - Error: "Collateral cap reached. Maximum allowed: {{maxQty}} {{token}}."
   - Cause: Deposit would exceed collateral limit
   - Solution: Reduce deposit amount

7. **Pending Transaction**
   - Error: "You have a pending transaction" / "You have pending transactions"
   - Cause: Previous deposit transaction still pending
   - Solution: Wait for pending transaction to complete

8. **Deposit Failed**
   - Error: "Deposit failed, please try again later."
   - Cause: Transaction failed on blockchain
   - Solution: Check transaction on block explorer, retry if needed

## Important Notes

1. **Network Selection**: User must select the correct network that matches their wallet's connected chain.

2. **Token Support**: Not all tokens are supported. Check available tokens for selected chain.

3. **Bridgeless Networks**: Use bridgeless networks when possible to avoid bridge fees.

4. **Swap Fees**: Swapping tokens incurs a 0.025% fee. Consider depositing USDC directly if possible.

5. **Gas Tokens**: Always maintain sufficient native tokens (ETH, BNB, etc.) for gas fees.

6. **Destination Gas**: Cross-chain deposits may require gas tokens on destination chain.

7. **Slippage**: Set appropriate slippage tolerance when swapping to avoid transaction reversals.

8. **Transaction Monitoring**: Monitor transaction status. Failed transactions may require retry.

9. **Minimum Amounts**: Each chain/token may have different minimum deposit amounts.

10. **Collateral Limits**: Some tokens have collateral caps. Check limits before depositing large amounts.

## Example: Complete Deposit Flow

### Mainnet - Direct Deposit (USDC on Arbitrum)

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
   - Shows gas fee estimate: ~0.001 ETH

4. Token Approval Check
   - Checks USDC allowance
   - If insufficient, shows "Approve" button
   - User clicks "Approve"
   - Signs approval transaction
   - Waits for confirmation: ~30 seconds

5. User Confirms Deposit
   - Reviews deposit details
   - Clicks "Deposit"
   - Wallet prompts for signature
   - User signs transaction
   - Transaction hash: 0xabc123...

6. Transaction Submitted
   - Status: "Deposit requested"
   - System monitors transaction
   - Waits for confirmation: ~1-2 minutes

7. Transaction Confirmed
   - Status: "Deposit completed"
   - 1000 USDC added to broker account
   - Available balance updated
   - Deposit history updated
```

### Testnet - Cross-Chain Deposit (ETH from Ethereum to Arbitrum)

```
1. User Opens Assets Page
   - networkId = "testnet"
   - API Base URL = "https://testnet-api.orderly.org"
   - User has 1 ETH on Ethereum Sepolia

2. User Clicks "Deposit"
   - Opens deposit dialog
   - Selects: Ethereum Sepolia (chain ID: 11155111)
   - Selects: ETH
   - Shows balance: 1 ETH

3. User Enters Amount: 0.5 ETH
   - System detects: Swap needed (ETH → USDC)
   - System detects: Bridge needed (Ethereum → Arbitrum)
   - Shows preview:
     - Swap: 0.5 ETH → ~1500 USDC (0.025% fee)
     - Bridge: ~1500 USDC (Ethereum → Arbitrum)
     - Bridge fee: ~5 USDC
     - Deposit: ~1495 USDC to broker

4. User Approves Swap
   - Clicks "Confirm to swap"
   - Signs swap transaction
   - Waits for confirmation: ~1 minute
   - Receives ~1500 USDC on Ethereum

5. User Approves Bridge
   - Clicks "Bridge"
   - Signs bridge transaction
   - Status: "Bridging..."
   - Waits for cross-chain: ~5-10 minutes
   - USDC arrives on Arbitrum Sepolia

6. User Approves Deposit
   - Clicks "Deposit"
   - Signs deposit transaction on Arbitrum
   - Status: "Depositing..."
   - Waits for confirmation: ~1 minute

7. Deposit Completed
   - Status: "Deposit completed"
   - ~1495 USDC added to broker account
   - Balance updated
```

## Debugging

### Check Deposit Status

```javascript
// In browser console or via API
GET /v1/asset/deposit?status=completed
```

### Check Transaction on Block Explorer

```javascript
// Get transaction hash from deposit history
// View on block explorer:
// Mainnet: https://etherscan.io/tx/{tx_hash}
// Testnet: https://sepolia.etherscan.io/tx/{tx_hash}
```

### Monitor Deposit Transactions

1. Open browser DevTools → Network tab
2. Filter by "orderly" or "asset"
3. Look for:
   - `GET /v1/asset/deposit` - Fetch deposit history
   - Monitor WebSocket connections for real-time updates

### Check Wallet Balance

```javascript
// In browser console (if wallet connected)
// Check token balance via wallet provider
```

## Summary

The deposit function in the Portfolio Assets page provides a comprehensive way to deposit assets from Web3 wallets to Orderly Network broker accounts. It handles:

1. **Direct Deposits**: Simple deposits on bridgeless networks
2. **Cross-Chain Bridging**: Via Stargate when bridging required
3. **Token Swapping**: Via WOOFi when converting tokens to USDC
4. **Token Approvals**: For ERC-20 tokens requiring approval
5. **Multi-Step Transactions**: Combines swap, bridge, and deposit when needed

The process is fully automated and guides users through each step, with clear fee disclosures and transaction status updates. All deposits work identically on both mainnet and testnet, with the only difference being the networks, contracts, and whether assets have real value.

