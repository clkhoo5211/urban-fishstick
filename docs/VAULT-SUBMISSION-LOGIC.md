# Vault Submission Logic - Mainnet & Testnet

This document explains how vault deposits and withdrawals work in both mainnet and testnet environments.

## Overview

The vault functionality is implemented using the `@orderly.network/vaults` package (version 2.8.1), which is a pre-built React component library that handles all vault operations. The actual submission logic is encapsulated within this package, but we can understand the architecture and flow from the codebase.

## Architecture

### Component Structure

```
App.tsx
  └── OrderlyProvider (app/components/orderlyProvider/index.tsx)
      └── OrderlyAppProvider (from @orderly.network/react-app)
          └── VaultsPage (from @orderly.network/vaults)
              └── VaultDepositAndWithdraw
                  ├── VaultDepositWidget
                  └── VaultWithdrawWidget
```

### Key Files

1. **Vault Page Entry**: `app/pages/vaults/Index.tsx`
   - Renders `<VaultsPage />` from `@orderly.network/vaults`

2. **Network Configuration**: `app/components/orderlyProvider/index.tsx`
   - Determines `networkId` (mainnet/testnet)
   - Passes to `OrderlyAppProvider`

3. **Vault Package**: `node_modules/@orderly.network/vaults/`
   - Contains all vault logic, UI components, and API calls

## Network Selection Logic

### How Network ID is Determined

The network (mainnet/testnet) is determined by the `getNetworkId()` function in `app/components/orderlyProvider/index.tsx`:

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
VaultsPage component (from @orderly.network/vaults) 
inherits network context from OrderlyAppProvider
    ↓
All vault API calls use the correct base URL
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

### Network Switching

When a user connects a wallet and switches chains:

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

## Vault API Endpoints

Based on the type definitions and typical Orderly Network API structure, the vault operations use the following endpoints:

### Mainnet Endpoints

- **Get Vault Info**: `GET https://api.orderly.org/v1/vault/info`
- **Get Vault LP Performance**: `GET https://api.orderly.org/v1/vault/lp/performance`
- **Get Vault LP Info**: `GET https://api.orderly.org/v1/vault/lp/info`
- **Submit Deposit/Withdrawal**: `POST https://api.orderly.org/v1/vault/operation`

### Testnet Endpoints

- **Get Vault Info**: `GET https://testnet-api.orderly.org/v1/vault/info`
- **Get Vault LP Performance**: `GET https://testnet-api.orderly.org/v1/vault/lp/performance`
- **Get Vault LP Info**: `GET https://testnet-api.orderly.org/v1/vault/lp/info`
- **Submit Deposit/Withdrawal**: `POST https://testnet-api.orderly.org/v1/vault/operation`

**Note**: The exact endpoint paths may vary. These are inferred from the API structure and type definitions.

## Vault Data Structures

### VaultInfo Interface

```typescript
interface VaultInfo {
  vault_id: string;
  vault_address: string;
  vault_type: string;
  performance_fee_rate: number;
  supported_chains: VaultSupportedChain[];
  tvl: number;
  apr30_d: number;
  vault_lifetime_net_pnl: number;
  lp_counts: number;
  min_deposit_amount: number;
  min_withdrawal_amount: number;
  total_main_shares: number;
  est_main_share_price: number;
  gate_threshold_pct: number;
  gate_triggered: boolean;
  lock_duration: number;
  broker_id: string;
  "30d_apr": number;
  "30d_apy": number;
}
```

### VaultOperationMessage Interface

```typescript
interface VaultOperationMessage {
  payloadType: string;      // Type of operation
  nonce: string;            // Unique nonce for the operation
  receiver: string;         // Receiver address
  amount: string;           // Amount in string format
  vaultId: string;          // Vault identifier
  token: string;            // Token symbol (e.g., "USDC")
  dexBrokerId: string;      // Broker ID
  chainId?: string;         // Optional chain ID
  chainType?: string;       // Optional chain type
}
```

### VaultOperationRequest Interface

```typescript
type VaultOperationRequest = {
  message: VaultOperationMessage;
  signature: string;              // User's signature for the message
  userAddress: string;            // User's wallet address
  verifyingContract: string;      // Contract address for signature verification
};
```

## Deposit Submission Flow

### Step-by-Step Process

1. **User Initiates Deposit**
   - User clicks on a vault card
   - Opens `VaultDepositAndWithdraw` dialog/sheet with `activeTab="deposit"`
   - `VaultDepositWidget` is rendered

2. **User Input**
   - User enters deposit amount
   - System validates:
     - Minimum deposit amount (`min_deposit_amount` from `VaultInfo`)
     - User's available balance
     - Account type (must be main account, not sub-account)

3. **Message Construction**
   - Creates `VaultOperationMessage`:
     ```typescript
     {
       payloadType: "deposit",
       nonce: generateNonce(),
       receiver: vault_address,
       amount: depositAmount.toString(),
       vaultId: vault_id,
       token: "USDC",
       dexBrokerId: brokerId,
       chainId: currentChainId,
       chainType: "evm" | "solana"
     }
     ```

4. **Signature Generation**
   - User signs the message using their wallet
   - Signature format depends on chain type:
     - **EVM**: EIP-712 typed data signature
     - **Solana**: Ed25519 signature

5. **Request Submission**
   - Constructs `VaultOperationRequest`:
     ```typescript
     {
       message: vaultOperationMessage,
       signature: userSignature,
       userAddress: userWalletAddress,
       verifyingContract: vaultContractAddress
     }
     ```

6. **API Call**
   - POST request to `/v1/vault/operation`
   - Base URL determined by `networkId`:
     - Mainnet: `https://api.orderly.org`
     - Testnet: `https://testnet-api.orderly.org`
   - Request includes authentication headers (if required)

7. **Response Handling**
   - Success: Operation queued/initiated
   - Error: Display error message to user
   - Common errors:
     - `vaults.operation.error.minDeposit`: Deposit below minimum
     - `vaults.operation.error.switchAccount`: Must use main account

8. **Status Tracking**
   - User can check operation status
   - Latest deposit shown in `LatestDepositWidget`

## Withdrawal Submission Flow

### Step-by-Step Process

1. **User Initiates Withdrawal**
   - User clicks on a vault card
   - Opens `VaultDepositAndWithdraw` dialog/sheet with `activeTab="withdraw"`
   - `VaultWithdrawWidget` is rendered

2. **User Input**
   - User enters withdrawal amount (in shares or USDC)
   - System calculates:
     - Estimated shares to withdraw
     - Estimated USDC to receive (based on `est_main_share_price`)
     - Validates minimum withdrawal amount

3. **Confirmation Dialog**
   - Shows withdrawal details
   - Displays estimated receiving amount
   - Note: "Estimated USDC is based on current price per share, final amount determined at vault period end"

4. **Message Construction**
   - Creates `VaultOperationMessage`:
     ```typescript
     {
       payloadType: "withdrawal",
       nonce: generateNonce(),
       receiver: userWalletAddress,
       amount: withdrawalAmount.toString(),
       vaultId: vault_id,
       token: "USDC",
       dexBrokerId: brokerId,
       chainId: currentChainId,
       chainType: "evm" | "solana"
     }
     ```

5. **Signature Generation**
   - Same process as deposit (EIP-712 or Ed25519)

6. **Request Submission**
   - Same API endpoint: `POST /v1/vault/operation`
   - Different `payloadType` in message

7. **Withdrawal Process**
   - **Initiate**: Withdrawal request submitted
   - **Vault Process**: Vault processes the withdrawal (can take up to 6 hours)
   - **Transferred**: Funds transferred to user

8. **Status Tracking**
   - User can check withdrawal status
   - Latest withdrawal shown in `LatestWithdrawWidget`

## Network-Specific Behavior

### Mainnet Behavior

- **Base URL**: `https://api.orderly.org`
- **Real Assets**: Uses real USDC
- **Real Transactions**: Actual on-chain transactions
- **Production Vaults**: Production vault contracts
- **Supported Chains**: From `VITE_ORDERLY_MAINNET_CHAINS` config

### Testnet Behavior

- **Base URL**: `https://testnet-api.orderly.org`
- **Test Assets**: Uses testnet USDC
- **Test Transactions**: Testnet chain transactions
- **Test Vaults**: Testnet vault contracts
- **Supported Chains**: From `VITE_ORDERLY_TESTNET_CHAINS` config

### Network Detection in Vault Components

The vault components from `@orderly.network/vaults` receive the network context through:

1. **OrderlyAppProvider Context**: Provides `networkId` to all child components
2. **API Client**: Automatically uses correct base URL based on `networkId`
3. **Chain Filter**: Only shows vaults supported on current network's chains

## Key Differences: Mainnet vs Testnet

| Aspect | Mainnet | Testnet |
|--------|---------|---------|
| **API Base URL** | `https://api.orderly.org` | `https://testnet-api.orderly.org` |
| **Assets** | Real USDC | Testnet USDC |
| **Transactions** | Real on-chain | Testnet on-chain |
| **Vault Contracts** | Production addresses | Testnet addresses |
| **Network ID** | `"mainnet"` | `"testnet"` |
| **Default** | Yes (if both enabled) | No (unless mainnet disabled) |
| **localStorage Key** | `"orderly_network_id"` | `"orderly_network_id"` |

## State Management

The vaults package uses Zustand for state management:

### VaultsStore State

```typescript
interface VaultsState {
  baseUrl: string;                    // API base URL (mainnet/testnet)
  vaultInfo: {
    data: VaultInfo[];
    loading: boolean;
    error: string | null;
    lastUpdated: number | null;
  };
  vaultLpPerformance: {
    data: Record<string, VaultLpPerformance[]>;
    loading: boolean;
    error: string | null;
    lastUpdated: number | null;
    params: VaultPerformanceParams | null;
  };
  vaultLpInfo: {
    data: Record<string, VaultLpInfo[]>;
    loading: boolean;
    error: string | null;
    lastUpdated: number | null;
    params: VaultLpInfoParams | null;
  };
  vaultsPageConfig: VaultsPageConfig | null;
}
```

### Available Hooks

- `useVaultInfoState()`: Get vault information
- `useVaultLpInfoState()`: Get user's LP information
- `useVaultLpPerformanceState()`: Get vault performance data
- `useVaultInfoActions()`: Fetch/refresh vault info
- `useVaultLpInfoActions()`: Fetch/refresh LP info
- `useVaultCardScript(vault)`: Get vault card data and actions

## Error Handling

### Common Errors

1. **Minimum Deposit Error**
   - Message: `"vaults.operation.error.minDeposit"`
   - Condition: Deposit amount < `min_deposit_amount`
   - Solution: Increase deposit amount

2. **Account Type Error**
   - Message: `"vaults.operation.error.switchAccount"`
   - Condition: User is on sub-account
   - Solution: Switch to main account

3. **Network Mismatch**
   - Condition: Wallet chain doesn't match network
   - Solution: Switch wallet to correct chain

4. **Insufficient Balance**
   - Condition: User balance < deposit amount
   - Solution: Deposit more funds to account

## Example: Complete Deposit Flow

```
1. User on Mainnet
   - networkId = "mainnet"
   - API Base URL = "https://api.orderly.org"

2. User Clicks Vault Card
   - Opens VaultDepositAndWithdraw dialog
   - VaultDepositWidget rendered

3. User Enters Amount: 1000 USDC
   - Validates: min_deposit_amount = 100 USDC ✓
   - Validates: user balance = 5000 USDC ✓
   - Validates: main account ✓

4. User Clicks Deposit
   - Message constructed:
     {
       payloadType: "deposit",
       nonce: "0x123...",
       receiver: "0xVaultAddress...",
       amount: "1000",
       vaultId: "orderly_omni_vault",
       token: "USDC",
       dexBrokerId: "demo",
       chainId: "42161",
       chainType: "evm"
     }

5. User Signs Message
   - EIP-712 signature generated
   - signature = "0xabc..."

6. Request Sent
   - POST https://api.orderly.org/v1/vault/operation
   - Body: {
       message: {...},
       signature: "0xabc...",
       userAddress: "0xUser...",
       verifyingContract: "0xVault..."
     }

7. Response Received
   - Success: Operation queued
   - User sees confirmation
   - Latest deposit updated
```

## Example: Complete Withdrawal Flow

```
1. User on Testnet
   - networkId = "testnet"
   - API Base URL = "https://testnet-api.orderly.org"

2. User Clicks Vault Card
   - Opens VaultDepositAndWithdraw dialog
   - VaultWithdrawWidget rendered

3. User Enters Amount: 500 USDC
   - Calculates: shares = 500 / est_main_share_price
   - Shows: Estimated receiving amount

4. User Confirms
   - Confirmation dialog shown
   - User clicks "Initiate Withdrawal"

5. Message Constructed
   {
     payloadType: "withdrawal",
     nonce: "0x456...",
     receiver: "0xUser...",
     amount: "500",
     vaultId: "orderly_omni_vault",
     token: "USDC",
     dexBrokerId: "demo",
     chainId: "421614",
     chainType: "evm"
   }

6. User Signs Message
   - EIP-712 signature generated

7. Request Sent
   - POST https://testnet-api.orderly.org/v1/vault/operation
   - Same request structure as deposit

8. Withdrawal Process
   - Status: "Initiate" → "Vault Process" → "Transferred"
   - Up to 6 hours for completion
```

## Important Notes

1. **Network Isolation**: Mainnet and testnet are completely separate. Operations on one don't affect the other.

2. **Network Switching**: When switching networks, the page reloads to ensure clean state.

3. **Signature Verification**: All operations require user signatures. The signature format depends on the blockchain (EVM vs Solana).

4. **Operation Nonce**: Each operation has a unique nonce to prevent replay attacks.

5. **Base URL Inheritance**: The vault components automatically use the correct API base URL based on the `networkId` from `OrderlyAppProvider`.

6. **Chain Support**: Vaults may support different chains on mainnet vs testnet. Check `supported_chains` in `VaultInfo`.

7. **Lock Duration**: Some vaults have a lock duration. Users cannot withdraw during the lock period.

8. **Price Per Share**: The `est_main_share_price` is an estimate. Final withdrawal amount is determined at vault period end.

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
// - VITE_ORDERLY_MAINNET_CHAINS
// - VITE_ORDERLY_TESTNET_CHAINS
```

### Monitor API Calls

1. Open browser DevTools → Network tab
2. Filter by "orderly" or "vault"
3. Look for:
   - `GET /v1/vault/info` - Fetching vault list
   - `GET /v1/vault/lp/info` - Fetching user's LP info
   - `POST /v1/vault/operation` - Submitting deposit/withdrawal

### Check Vault Store State

The vaults package uses Zustand store. You can inspect it if needed, though the internal implementation is in the compiled package.

## Summary

The vault submission logic works identically for both mainnet and testnet, with the only difference being:

1. **API Base URL**: Determined by `networkId` from `OrderlyAppProvider`
2. **Chain IDs**: Different chains supported on each network
3. **Contracts**: Different vault contract addresses
4. **Assets**: Real vs testnet tokens

The actual submission logic is handled by the `@orderly.network/vaults` package, which:
- Constructs operation messages
- Handles wallet signatures
- Submits to the correct API endpoint
- Manages operation status
- Displays results to users

All network-specific logic is abstracted away through the `OrderlyAppProvider` context, making the vault components network-agnostic.

