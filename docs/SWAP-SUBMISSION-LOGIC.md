# Swap Submission Logic - Mainnet & Testnet

This document explains how the swap functionality works in both mainnet and testnet environments.

## Overview

The swap functionality is implemented using the **WooFi Swap Widget Kit** (`woofi-swap-widget-kit` version 0.0.12), which is a third-party React component library that provides a complete swap interface. The widget handles all swap operations, token selection, price quotes, and transaction execution.

## Architecture

### Component Structure

```
App.tsx
  └── OrderlyProvider (app/components/orderlyProvider/index.tsx)
      └── OrderlyAppProvider (from @orderly.network/react-app)
          └── SwapLayout (app/pages/swap/Layout.tsx)
              └── SwapIndex (app/pages/swap/Index.tsx)
                  └── WooFiWidget (app/components/WooFiWidget.tsx)
                      └── WooFiSwapWidgetReact (from woofi-swap-widget-kit/react)
```

### Key Files

1. **Swap Page Entry**: `app/pages/swap/Index.tsx`
   - Renders `<WooFiWidget />` component
   - Wrapped in Suspense for lazy loading

2. **Swap Layout**: `app/pages/swap/Layout.tsx`
   - Provides Scaffold navigation structure
   - Sets initial menu to `/swap`

3. **WooFi Widget Wrapper**: `app/components/WooFiWidget.tsx`
   - Integrates WooFi widget with Orderly wallet connector
   - Handles wallet connection and chain switching

4. **WooFi Package**: `node_modules/woofi-swap-widget-kit/`
   - Third-party swap widget library
   - Contains all swap logic, UI, and API calls

5. **Styling**: `app/styles/woofi-widget.css`
   - Custom CSS overrides to match Orderly theme
   - Integrates with Orderly UI color variables

## Network Selection Logic

### How Network is Determined

The swap widget uses the **connected chain** from the wallet, not the Orderly `networkId`. The network is determined by:

1. **Connected Wallet Chain**: The chain the user's wallet is connected to
2. **Chain Filter**: Based on `VITE_ORDERLY_MAINNET_CHAINS` or `VITE_ORDERLY_TESTNET_CHAINS`
3. **Network Detection**: The widget internally detects if the chain is mainnet or testnet

### Network Configuration Flow

```
User Connects Wallet
    ↓
OrderlyAppProvider detects networkId (mainnet/testnet)
    ↓
Wallet Connector initializes with chain filter:
    - Mainnet chains: From VITE_ORDERLY_MAINNET_CHAINS
    - Testnet chains: From VITE_ORDERLY_TESTNET_CHAINS
    ↓
WooFiWidget receives:
    - evmProvider: wallet?.provider (from useWalletConnector)
    - currentChain: connectedChain?.id (from useWalletConnector)
    ↓
WooFiSwapWidgetReact uses connected chain
    - Detects network based on chain ID
    - Uses appropriate swap routes/contracts
    - Handles mainnet/testnet swaps automatically
```

### Runtime Configuration

**File**: `public/config.js`

```javascript
{
  "VITE_DISABLE_MAINNET": "false",
  "VITE_DISABLE_TESTNET": "false",
  "VITE_ORDERLY_MAINNET_CHAINS": "1,56,43114,900900900,42161,10,1329,146,1514,80094,8453,5000,34443,2818,98866,2741",
  "VITE_ORDERLY_TESTNET_CHAINS": "97,421614,901901901,10143,11124",
  "VITE_DEFAULT_CHAIN": ""
}
```

### Chain-Based Network Detection

The swap widget determines network by checking the connected chain ID:

**Mainnet Chains** (from config):
- Ethereum: 1
- BNB Chain: 56
- Avalanche: 43114
- Arbitrum: 42161
- Optimism: 10
- Base: 8453
- And more...

**Testnet Chains** (from config):
- BNB Testnet: 97
- Arbitrum Sepolia: 421614
- And more...

## WooFi Widget Integration

### Component Implementation

**File**: `app/components/WooFiWidget.tsx`

```typescript
export default function WooFiWidget() {
  const { wallet, setChain, connectedChain, connect } = useWalletConnector();

  const handleConnectWallet = useCallback(() => {
    connect();
  }, [connect]);

  const handleChainSwitch = useCallback(
    (targetChain: { chainName: string; chainId?: string; key: string }) => {
      if (targetChain.chainId) {
        setChain({ chainId: Number(targetChain.chainId) });
      }
    },
    [setChain]
  );

  return (
    <WooFiSwapWidgetReact
      evmProvider={wallet?.provider}        // Wallet provider for transactions
      currentChain={connectedChain?.id}     // Current chain ID
      onConnectWallet={handleConnectWallet} // Wallet connection handler
      onChainSwitch={handleChainSwitch}     // Chain switching handler
    />
  );
}
```

### Props Explained

1. **`evmProvider`**: The wallet provider (MetaMask, WalletConnect, etc.)
   - Used for signing transactions
   - Passed from `useWalletConnector()` hook
   - `undefined` if wallet not connected

2. **`currentChain`**: Current connected chain ID
   - Used to determine network (mainnet/testnet)
   - Used to filter available tokens
   - Updates when user switches chains

3. **`onConnectWallet`**: Callback when user clicks "Connect Wallet"
   - Triggers Orderly wallet connection flow
   - Opens wallet selection modal

4. **`onChainSwitch`**: Callback when user switches chains in widget
   - Receives target chain info from widget
   - Calls `setChain()` to switch wallet chain
   - Widget updates automatically

## Swap Flow

### Step-by-Step Process

1. **User Opens Swap Page**
   - Navigates to `/swap`
   - `SwapIndex` component renders
   - `WooFiWidget` lazy loads

2. **Widget Initialization**
   - `WooFiSwapWidgetReact` initializes
   - Checks if wallet is connected
   - Detects current chain from `currentChain` prop
   - Loads available tokens for the chain

3. **Token Selection**
   - User selects "From" token
   - User selects "To" token
   - Widget fetches available tokens based on chain
   - Token list may differ for mainnet vs testnet

4. **Amount Input**
   - User enters swap amount
   - Widget calculates:
     - Output amount
     - Price impact
     - Estimated gas fees
     - Route information

5. **Price Quote**
   - Widget fetches quote from WooFi API
   - Quote includes:
     - Output amount
     - Exchange rate
     - Price impact
     - Estimated fees
   - Quote is network-specific (mainnet/testnet)

6. **Transaction Preparation**
   - If wallet not connected, prompts connection
   - If wrong chain, prompts chain switch
   - Validates:
     - Sufficient balance
     - Token approval (if needed)
     - Minimum amount requirements

7. **Token Approval** (if needed)
   - For ERC-20 tokens, approval required first
   - User signs approval transaction
   - Transaction sent to blockchain
   - Waits for confirmation

8. **Swap Transaction**
   - User reviews swap details
   - Clicks "Swap" button
   - Wallet prompts for transaction signature
   - User signs transaction
   - Transaction submitted to blockchain

9. **Transaction Confirmation**
   - Widget monitors transaction status
   - Shows pending state
   - Waits for blockchain confirmation
   - Updates UI on success/failure

10. **Completion**
    - Swap completed successfully
    - User receives output tokens
    - Transaction hash displayed
    - User can view on block explorer

## Network-Specific Behavior

### Mainnet Behavior

- **Real Assets**: Swaps real tokens (USDC, ETH, etc.)
- **Real Transactions**: Actual on-chain transactions
- **Production Contracts**: Uses mainnet swap contracts
- **Gas Fees**: Real ETH/gas costs
- **Block Explorer**: Links to mainnet explorer (Etherscan, etc.)
- **Supported Chains**: From `VITE_ORDERLY_MAINNET_CHAINS` config

### Testnet Behavior

- **Test Assets**: Swaps testnet tokens
- **Test Transactions**: Testnet chain transactions
- **Test Contracts**: Uses testnet swap contracts
- **Free Gas**: Testnet usually has free/cheap gas
- **Test Explorer**: Links to testnet explorer
- **Supported Chains**: From `VITE_ORDERLY_TESTNET_CHAINS` config

### Chain-Based Network Detection

The WooFi widget automatically detects network based on chain ID:

```typescript
// Mainnet chains (examples)
1   → Ethereum Mainnet
56  → BNB Chain Mainnet
42161 → Arbitrum Mainnet
8453 → Base Mainnet

// Testnet chains (examples)
97 → BNB Chain Testnet
421614 → Arbitrum Sepolia
```

## Wallet Integration

### useWalletConnector Hook

The widget uses `useWalletConnector()` from `@orderly.network/hooks`:

```typescript
const { 
  wallet,          // Wallet instance
  setChain,        // Function to switch chains
  connectedChain,  // Current connected chain info
  connect          // Function to connect wallet
} = useWalletConnector();
```

### Wallet Connection Flow

1. **User Clicks "Connect Wallet"**
   - Widget calls `onConnectWallet` callback
   - `handleConnectWallet()` calls `connect()`
   - Orderly wallet modal opens
   - User selects wallet (MetaMask, WalletConnect, etc.)

2. **Wallet Connection**
   - Wallet connects to user's account
   - `wallet` object populated
   - `connectedChain` updated
   - `evmProvider` passed to widget

3. **Chain Detection**
   - Widget reads `currentChain` prop
   - Determines network (mainnet/testnet)
   - Loads appropriate token list

### Chain Switching Flow

1. **User Switches Chain in Widget**
   - Widget detects chain switch request
   - Calls `onChainSwitch` callback
   - `handleChainSwitch()` receives target chain
   - Calls `setChain({ chainId: Number(targetChain.chainId) })`

2. **Wallet Chain Switch**
   - Wallet prompts user to switch chain
   - User approves switch in wallet
   - Wallet switches to target chain
   - `connectedChain` updates
   - Widget re-initializes with new chain

## WooFi API Integration

The WooFi widget makes API calls to WooFi's backend services. These are handled internally by the widget, but we can infer the structure:

### Typical API Endpoints

**Token List**:
- `GET https://api.woofi.network/v1/tokens?chain={chainId}`
- Returns available tokens for the chain

**Price Quote**:
- `GET https://api.woofi.network/v1/quote?from={token}&to={token}&amount={amount}&chain={chainId}`
- Returns swap quote with output amount, price impact, etc.

**Swap Route**:
- `GET https://api.woofi.network/v1/route?from={token}&to={token}&amount={amount}&chain={chainId}`
- Returns optimal swap route

**Transaction Building**:
- `POST https://api.woofi.network/v1/swap`
- Builds swap transaction data

**Note**: These endpoints are inferred from typical DEX aggregator patterns. The actual endpoints may vary.

### Network-Specific API Calls

The widget automatically uses the correct API endpoints based on the connected chain:

- **Mainnet chains** → Mainnet API endpoints
- **Testnet chains** → Testnet API endpoints

## Transaction Flow

### ERC-20 Token Approval

For ERC-20 tokens, approval is required before swapping:

```typescript
// 1. User initiates swap
// 2. Widget checks allowance
if (allowance < amount) {
  // 3. Approval transaction needed
  const approveTx = {
    to: tokenAddress,
    data: approveFunctionCall(spenderAddress, amount),
    value: "0x0"
  };
  
  // 4. User signs approval
  await wallet.provider.request({
    method: 'eth_sendTransaction',
    params: [approveTx]
  });
  
  // 5. Wait for approval confirmation
  await waitForTransaction(approveTxHash);
}
```

### Swap Transaction

After approval (if needed), the swap transaction is executed:

```typescript
// 1. Build swap transaction
const swapTx = {
  to: swapRouterAddress,
  data: swapFunctionCall(fromToken, toToken, amount, minAmount),
  value: isNativeToken ? amount : "0x0"
};

// 2. User signs swap transaction
const txHash = await wallet.provider.request({
  method: 'eth_sendTransaction',
  params: [swapTx]
});

// 3. Wait for confirmation
await waitForTransaction(txHash);

// 4. Swap completed
```

## Error Handling

### Common Errors

1. **Wallet Not Connected**
   - Widget shows "Connect Wallet" button
   - User clicks to connect

2. **Wrong Chain**
   - Widget detects chain mismatch
   - Shows "Switch Chain" button
   - User switches to supported chain

3. **Insufficient Balance**
   - Widget checks user balance
   - Shows error if balance < swap amount
   - User needs to deposit more tokens

4. **Insufficient Allowance**
   - Widget checks token allowance
   - Prompts for approval if needed
   - User approves token

5. **Slippage Exceeded**
   - Price moved during transaction
   - Transaction reverts
   - User can retry with higher slippage tolerance

6. **Transaction Failed**
   - Transaction reverts on blockchain
   - Widget shows error message
   - User can retry

## Styling Integration

### Custom CSS Overrides

**File**: `app/styles/woofi-widget.css`

The widget is styled to match Orderly's theme using CSS variables:

```css
/* Button styling */
.el-button.el-button--primary {
  background: rgb(var(--oui-color-primary)) !important;
  border-color: rgb(var(--oui-color-primary)) !important;
  color: rgb(var(--oui-color-primary-contrast)) !important;
}

/* Swap container */
.dex .swap {
  background: linear-gradient(135deg, 
    rgb(var(--oui-gradient-primary-start)) 0%, 
    rgb(var(--oui-gradient-primary-end)) 100%) !important;
  border-radius: var(--oui-rounded-lg) !important;
}

/* Input fields */
.coin-input .input-main {
  background: rgb(var(--oui-color-base-3)) !important;
  border-radius: var(--oui-rounded-xl) !important;
}
```

### Theme Variables Used

- `--oui-color-primary`: Primary brand color
- `--oui-color-primary-contrast`: Contrast color for primary
- `--oui-color-primary-light`: Light variant of primary
- `--oui-color-base-3`: Base background color
- `--oui-color-base-8`: Elevated background color
- `--oui-color-base-foreground`: Text color
- `--oui-color-line`: Border color
- `--oui-gradient-primary-start`: Gradient start
- `--oui-gradient-primary-end`: Gradient end
- `--oui-rounded-lg`: Large border radius
- `--oui-rounded-xl`: Extra large border radius

## Key Differences: Mainnet vs Testnet

| Aspect | Mainnet | Testnet |
|--------|---------|---------|
| **Network Detection** | Based on chain ID (1, 56, 42161, etc.) | Based on chain ID (97, 421614, etc.) |
| **Assets** | Real tokens (USDC, ETH, etc.) | Testnet tokens |
| **Transactions** | Real on-chain | Testnet on-chain |
| **Gas Fees** | Real costs (ETH/gas) | Free/cheap (testnet) |
| **Contracts** | Production swap contracts | Testnet swap contracts |
| **Block Explorer** | Mainnet explorer (Etherscan) | Testnet explorer |
| **API Endpoints** | Mainnet WooFi API | Testnet WooFi API |
| **Token Lists** | Mainnet token list | Testnet token list |

## Important Notes

1. **Network Independence**: The swap widget operates independently from Orderly's `networkId`. It uses the connected wallet's chain to determine network.

2. **Chain-Based Detection**: Network (mainnet/testnet) is determined by the chain ID, not by Orderly's network configuration.

3. **Wallet Provider Required**: The widget requires a connected wallet with an EVM provider to function.

4. **Third-Party Service**: Swap functionality is provided by WooFi, not Orderly Network. The widget is a separate service.

5. **Chain Filtering**: The widget respects the chain filter from `VITE_ORDERLY_MAINNET_CHAINS` and `VITE_ORDERLY_TESTNET_CHAINS`, but this is handled at the wallet connector level.

6. **No Direct API Integration**: The swap widget doesn't directly use Orderly Network APIs. It uses WooFi's swap aggregation service.

7. **Automatic Network Detection**: The widget automatically detects whether the connected chain is mainnet or testnet based on the chain ID.

8. **Token Approval**: ERC-20 tokens require approval before swapping. The widget handles this automatically.

9. **Slippage Protection**: The widget includes slippage protection to prevent unfavorable swaps.

10. **Route Optimization**: The widget automatically finds the best swap route across multiple DEXs.

## Example: Complete Swap Flow

### Mainnet Swap (Ethereum)

```
1. User Connects Wallet
   - Chain: Ethereum Mainnet (chain ID: 1)
   - Wallet: MetaMask
   - evmProvider: MetaMask provider

2. Widget Initializes
   - Detects chain ID: 1 (mainnet)
   - Loads mainnet token list
   - Shows available tokens

3. User Selects Tokens
   - From: USDC
   - To: ETH
   - Amount: 1000 USDC

4. Widget Fetches Quote
   - API: WooFi mainnet API
   - Returns: ~0.4 ETH (example)
   - Price impact: 0.1%
   - Route: USDC → ETH (direct)

5. User Approves (if needed)
   - Checks USDC allowance
   - If insufficient, prompts approval
   - User signs approval transaction
   - Waits for confirmation

6. User Confirms Swap
   - Reviews swap details
   - Clicks "Swap"
   - Wallet prompts for signature
   - User signs transaction

7. Transaction Submitted
   - Transaction hash: 0xabc...
   - Status: Pending
   - Widget shows loading state

8. Transaction Confirmed
   - Block confirmation received
   - Swap completed
   - User receives ETH
   - Transaction visible on Etherscan
```

### Testnet Swap (Arbitrum Sepolia)

```
1. User Connects Wallet
   - Chain: Arbitrum Sepolia (chain ID: 421614)
   - Wallet: MetaMask
   - evmProvider: MetaMask provider

2. Widget Initializes
   - Detects chain ID: 421614 (testnet)
   - Loads testnet token list
   - Shows available testnet tokens

3. User Selects Tokens
   - From: Test USDC
   - To: Test ETH
   - Amount: 1000 Test USDC

4. Widget Fetches Quote
   - API: WooFi testnet API
   - Returns: ~0.4 Test ETH
   - Price impact: 0.1%
   - Route: Test USDC → Test ETH

5. User Approves (if needed)
   - Same approval flow as mainnet
   - Uses testnet tokens

6. User Confirms Swap
   - Same confirmation flow
   - Uses testnet contracts

7. Transaction Submitted
   - Transaction on Arbitrum Sepolia
   - Free/cheap gas fees
   - Fast confirmation

8. Transaction Confirmed
   - Block confirmation received
   - Swap completed
   - User receives Test ETH
   - Transaction visible on testnet explorer
```

## Debugging

### Check Current Chain

```javascript
// In browser console (after connecting wallet)
// The chain is managed by the wallet connector
// Check via React DevTools or widget state
```

### Check Wallet Connection

```javascript
// The wallet state is managed by useWalletConnector()
// Check if wallet is connected:
// - Wallet should be connected
// - evmProvider should exist
// - connectedChain should have id
```

### Monitor Network Requests

1. Open browser DevTools → Network tab
2. Filter by "woofi" or "swap"
3. Look for:
   - Token list requests
   - Quote requests
   - Route requests
   - Transaction building requests

### Check Transaction Status

1. Get transaction hash from widget
2. Visit block explorer:
   - Mainnet: Etherscan, BscScan, etc.
   - Testnet: Sepolia Etherscan, Testnet BscScan, etc.
3. Check transaction details

## Summary

The swap functionality is provided by the WooFi Swap Widget Kit, which is a third-party component that:

1. **Network Detection**: Automatically detects mainnet/testnet based on connected chain ID
2. **Wallet Integration**: Uses Orderly's wallet connector for wallet connection and chain switching
3. **Swap Execution**: Handles all swap logic, token approval, and transaction execution
4. **Network-Specific**: Automatically uses appropriate APIs, contracts, and token lists for mainnet vs testnet
5. **Theme Integration**: Styled to match Orderly's design system using CSS variables

The widget operates independently from Orderly's `networkId` and relies on the connected wallet's chain to determine the network environment. This allows users to swap tokens on any supported chain (mainnet or testnet) without needing to change Orderly's network configuration.

