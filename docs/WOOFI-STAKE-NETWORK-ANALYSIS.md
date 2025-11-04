# WooFi Stake - Network Fetching & Posting Analysis

This document analyzes how [WooFi's stake page](https://woofi.com/swap/stake) fetches and posts data, including all API endpoints, request/response structures, and data flow patterns.

## Overview

WooFi's stake page allows users to stake WOO tokens to earn rewards, participate in quests, and use boosters to increase their XP APR. The page displays staking statistics, quest campaigns, booster information, and user-specific staking data.

## Key API Endpoints

### Base API URLs
- **WooFi API**: `https://api.woofi.com/`
- **Binance API**: `https://api.binance.com/` (price data)
- **WooFi CDN**: `https://oss.woo.org/` (images, videos, and assets)
- **Third-Party**: CoinGecko (via WooFi proxy), 1inch (via WooFi proxy)

## Network Requests Breakdown

### 1. Initial Page Load Requests

#### GET Requests (Data Fetching)

**1. Staking Information (Main Data)**
```
GET https://api.woofi.com/stakingv2
```
- **Purpose**: Fetch overall staking statistics and APR information
- **Timing**: Initial page load
- **Method**: XMLHttpRequest
- **Response**: Staking statistics including total staked, average APR, base APR, and boosted APR

**2. Reward Distribution**
```
GET https://api.woofi.com/reward_distribution
```
- **Purpose**: Fetch weekly reward distribution statistics
- **Timing**: Initial page load
- **Method**: XMLHttpRequest
- **Response**: Weekly staking rewards amount

**3. Surge Campaign Information**
```
GET https://api.woofi.com/surge/infos
```
- **Purpose**: Fetch surge campaign details (special staking events)
- **Timing**: Initial page load
- **Method**: XMLHttpRequest
- **Response**: Array of surge campaign objects with rewards, duration, tasks, and status

**4. NFT Campaign Information (Quests)**
```
GET https://api.woofi.com/nft/campaign_infos
```
- **Purpose**: Fetch quest/NFT campaign information
- **Timing**: Initial page load, updated periodically
- **Method**: XMLHttpRequest
- **Response**: Array of quest campaign objects with tasks, rewards, and status

**5. Shared Configuration (Same as Earn Page)**
```
GET https://api.woofi.com/okx_tokens
GET https://api.woofi.com/1inch_tokens?network={network}
GET https://api.woofi.com/coingecko/coins_markets?page={page}
GET https://api.binance.com/api/v3/ticker/price?symbol=WOOUSDT
```
- **Purpose**: Token lists, market data, and price information
- **Timing**: Initial page load
- **Method**: XMLHttpRequest/Fetch

#### POST Requests

**1. Analytics Tracking**
```
POST https://api2.amplitude.com/2/httpapi
POST https://www.google-analytics.com/g/collect
```
- **Purpose**: User analytics and event tracking
- **Timing**: On user interactions and page events
- **Method**: POST

### 2. Asset Loading

**Images and Videos**:
- Booster NFTs: `https://oss.woo.org/static/images/nft/` (GIFs and WebM videos)
- Surge campaign images: `https://oss.woo.org/static/images/stake_surge_{token}/`
- Quest images: `https://oss.woo.org/static/images/nft/`
- Network logos: `https://oss.woo.org/static/images/sdk/`

## API Response Structures

### Staking Information Response (`/stakingv2`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": {
    "avg_apr": 8.01987685143261,
    "base_apr": 5.379612883248899,
    "mp_boosted_apr": 2.6402639681837115,
    "total_woo_staked": "579816827384741642706724342",
    "woo_decimals": 18
  }
}
```

**Key Fields**:
- `avg_apr`: Average Annual Percentage Rate across all stakers
- `base_apr`: Base APR without boosts
- `mp_boosted_apr`: Market Maker boosted APR component
- `total_woo_staked`: Total WOO tokens staked (in wei/raw units)
- `woo_decimals`: WOO token decimals (18)

**Calculation**:
- Total staked in human-readable format: `total_woo_staked / (10 ** woo_decimals)`
- Example: `579816827384741642706724342 / 10^18 = 579,816,827.38 WOO`

### Reward Distribution Response (`/reward_distribution`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": {
    "reward_distribution_weekly": 32134.700661400522
  }
}
```

**Key Fields**:
- `reward_distribution_weekly`: Weekly staking rewards distributed in USD

### Surge Campaign Response (`/surge/infos`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": [
    {
      "surge_id": "5",
      "network": "arbitrum",
      "title": "Grow your WOO: Season Two",
      "surge_title": "Grow your WOO: Season Two $WOO",
      "surge_description": "Increase your staked WOO amount to earn your share of 300,000 WOO in rewards!",
      "title_image": "https://oss.woo.org/static/images/stake_surge_woo/surge_woo_title.png",
      "upper_right_image": "https://oss.woo.org/static/images/stake_surge_woo/surge_woo_upper_right_image.png",
      "upper_right_image_mobile": "https://oss.woo.org/static/images/stake_surge_woo/surge_woo_upper_right_image_mobile.png",
      "description": "Increase your staked WOO amount to earn your share of 300,000 WOO in rewards!",
      "links": {
        "website": "https://woofi.com/about",
        "twitter": "https://x.com/_WOOFi",
        "learn_more": "",
        "detailed_rules": ""
      },
      "duration": {
        "start_time": "2025-10-03T00:00:00Z",
        "end_time": "2025-12-02T00:00:00Z",
        "claim_open_time": "2025-12-02T00:00:00Z",
        "claim_end_time": "2026-01-01T00:00:00Z",
        "claim_opened": false,
        "surge_status": "ONGOING",
        "surge_status_code": 1
      },
      "total_rewards": {
        "amount": 300000,
        "token": "0xcAFcD85D8ca7Ad1e1C6F82F651fA15E33AEfD07b",
        "symbol": "WOO",
        "decimals": 18,
        "token_image": "https://oss.woo.org/static/images/stake_surge_woo/surge_woo_token.png",
        "symbol_price": 0.0382797551276
      },
      "multiplier_tasks": [
        {
          "id": "55",
          "multiplier": 1.5,
          "button": "Boost",
          "button_link": "https://woofi.com/swap/stake",
          "title": "Stake a booster",
          "description": "Stake a booster"
        },
        {
          "id": "56",
          "multiplier": 2.0,
          "button": "Stake WOO",
          "button_link": "https://woofi.com/swap/stake",
          "title": "Earn XX/XXX XP",
          "description": "Earn XX/XXX XP"
        }
      ],
      "third_party_links": {
        "url": "",
        "image": "",
        "image_mobile": ""
      },
      "help_page_links": {
        "image1": "https://oss.woo.org/static/images/stake_surge_woo/step1.png",
        "image2": "https://oss.woo.org/static/images/stake_surge_woo/step2.png",
        "image3": "https://oss.woo.org/static/images/stake_surge_woo/step3.png"
      },
      "claim_contract_address": "",
      "woo_surge": true,
      "increased_woo": 6890720.0,
      "total_surge_score": "14039200.0"
    }
    // ... more surge campaigns
  ]
}
```

**Key Fields**:
- `surge_id`: Unique campaign identifier
- `network`: Network where the campaign is active
- `title` / `surge_title`: Campaign title
- `description`: Campaign description
- `duration`: Campaign timing information
  - `start_time` / `end_time`: Campaign period
  - `claim_open_time` / `claim_end_time`: Claim period
  - `surge_status`: Status (ONGOING, CLAIMS_OPEN, CLAIMS_END, etc.)
  - `surge_status_code`: Numeric status code
- `total_rewards`: Reward token information
  - `amount`: Total reward amount
  - `token`: Token contract address
  - `symbol`: Token symbol
  - `symbol_price`: Token price in USD
- `multiplier_tasks`: Tasks that provide score multipliers
- `increased_woo`: Amount of WOO increased during campaign
- `total_surge_score`: Total surge score across all participants

**Surge Status Codes**:
- `1`: ONGOING
- `2`: CLAIMS_OPEN
- `3`: CLAIMS_END
- `4`: ENDED

### NFT Campaign/Quest Response (`/nft/campaign_infos`)

**Response Structure**:
```json
{
  "status": "ok",
  "data": [
    {
      "quest_type": "mystery_box",
      "nft_tiers": {
        "epic": 2,
        "rare": 5,
        "uncommon": 25
      },
      "image_url": "https://oss.woo.org/static/images/nft/card_rotation.webm",
      "title": "Order UP!",
      "start_time": "Oct 21, 2025, UTC 00:00",
      "end_time": "Oct 24, 2025, UTC 00:00",
      "campaign_id": "22",
      "user_cap": -1,
      "task_ids": ["57", "58", "19"],
      "task_dependencies": {
        "19": ["57", "58"]
      },
      "token_reward_network": "arbitrum",
      "token_reward": "0x4E200fE2f3eFb977d5fd9c430A41531FB04d97B8",
      "token_name": "ORDER",
      "token_name_short": "ORDER",
      "token_reward_amount": "3000",
      "token_reward_count": 5,
      "token_reward_icon_url": "https://oss.woo.org/static/images/sdk/order.png",
      "quest_token_reward_manager": "0xca8EdCCF471A213cfd70D73117Ac7F49Bfc00B72",
      "note": "",
      "task_infos": {
        "57": {
          "display_order": 1,
          "description": "Task 1: Deposit ≥$25 into WOOFi Pro(EVM chains only)",
          "description_html": "Deposit ≥$25 into <a href=\"https://woofi.com/\" target=\"_blank\">WOOFi Pro</a>(EVM chains only)"
        },
        "58": {
          "display_order": 2,
          "description": "Trade ≥$10,000 on any pair via WOOFi Pro",
          "description_html": "Trade ≥$10,000 on any pair via <a href=\"https://woofi.com/\" target=\"_blank\">WOOFi Pro</a>"
        },
        "19": {
          "display_order": 3,
          "description": "Stake one Booster of any rarity",
          "description_html": "Stake one Booster of any rarity",
          "is_bonus": true,
          "user_tip": "This task will only be activated after completing Tasks 1 and 2..."
        }
      },
      "campaign_status_code": 3,
      "campaign_status": "END",
      "verified_nums": 104,
      "token_ids": {
        "uncommon": 1,
        "rare": 2,
        "epic": 3
      }
    }
    // ... more quest campaigns
  ]
}
```

**Key Fields**:
- `quest_type`: Type of quest (e.g., "mystery_box")
- `nft_tiers`: NFT tier distribution (epic, rare, uncommon counts)
- `title`: Quest title
- `start_time` / `end_time`: Quest period
- `campaign_id`: Unique campaign identifier
- `user_cap`: Maximum participants (-1 for unlimited)
- `task_ids`: Array of task IDs
- `task_dependencies`: Task dependency mapping (bonus tasks depend on others)
- `token_reward`: Optional token reward information
  - `token_reward_network`: Network for token reward
  - `token_reward`: Token contract address
  - `token_reward_amount`: Reward amount per winner
  - `token_reward_count`: Number of winners
- `task_infos`: Detailed task information
  - `display_order`: Display order
  - `description`: Task description
  - `description_html`: HTML-formatted description with links
  - `is_bonus`: Whether task is a bonus task
  - `user_tip`: Additional user tip
- `campaign_status_code`: Numeric status code
- `campaign_status`: Status text (END, ONGOING, etc.)
- `verified_nums`: Number of verified participants
- `token_ids`: NFT token ID mapping by tier

**Quest Status Codes**:
- `1`: ONGOING
- `2`: CLAIMS_OPEN
- `3`: END
- `4`: UPCOMING

## Data Fetching Flow

### Initial Page Load Sequence

```
1. Page HTML Loads
   ↓
2. JavaScript Bundles Load
   ↓
3. Initial API Calls (Parallel):
   - GET /stakingv2 (staking statistics)
   - GET /reward_distribution (weekly rewards)
   - GET /surge/infos (surge campaigns)
   - GET /nft/campaign_infos (quest campaigns)
   ↓
4. Shared Configuration (Parallel):
   - GET /okx_tokens
   - GET /1inch_tokens?network={network} (multiple networks)
   - GET /coingecko/coins_markets?page={page} (multiple pages)
   - GET /api.binance.com/api/v3/ticker/price?symbol=WOOUSDT
   ↓
5. Asset Loading:
   - Booster NFT images/videos
   - Surge campaign images
   - Quest images
   - Network logos
```

### Real-Time Updates

**Polling Pattern**:
- Staking statistics: Updated periodically (every 30-60 seconds)
- Reward distribution: Updated periodically
- Surge campaigns: Updated periodically
- Quest campaigns: Updated periodically
- Price data: Updated frequently (every 5-10 seconds)

## POST Request Patterns

### Stake/Unstake Flow (Inferred)

When users interact with staking, the following flow likely occurs:

```
1. User Connects Wallet
   ↓
2. Frontend Fetches User Position
   - Blockchain RPC calls (via Web3 provider)
   - Gets user's staked balance
   - Gets user's claimable rewards (USDC)
   - Gets user's XP
   - Gets user's booster count
   ↓
3. User Enters Stake Amount
   ↓
4. Frontend Validates Amount
   - Checks balance
   - Validates minimum stake
   ↓
5. User Approves Token (if needed)
   - ERC-20 approve transaction
   ↓
6. User Approves Stake Transaction
   - Wallet prompts for signature
   ↓
7. Transaction Submission
   - Smart contract interaction
   - Stakes WOO tokens
   ↓
8. Transaction Confirmation
   - Polls blockchain for confirmation
   - Updates UI
   ↓
9. Refresh Data
   - GET /stakingv2 (updates total staked)
   - Fetch user position (updates balance)
```

### Claim Rewards Flow (Inferred)

```
1. User Clicks "Claim" Button
   ↓
2. Frontend Checks Claimable Rewards
   - Blockchain RPC calls (via Web3 provider)
   - Gets claimable USDC rewards
   ↓
3. User Approves Claim Transaction
   - Wallet prompts for signature
   ↓
4. Transaction Submission
   - Smart contract interaction
   - Claims USDC rewards
   ↓
5. Transaction Confirmation
   - Polls blockchain for confirmation
   - Updates UI
   ↓
6. Refresh Data
   - GET /reward_distribution (updates weekly rewards)
   - Fetch user position (updates claimable balance)
```

### Compound Rewards Flow (Inferred)

```
1. User Clicks "Compound" Button
   ↓
2. Frontend Checks Claimable Rewards
   - Blockchain RPC calls (via Web3 provider)
   - Gets claimable USDC rewards
   ↓
3. Frontend Prepares Compound Transaction
   - Claims rewards and stakes them
   - Single transaction or batch
   ↓
4. User Approves Compound Transaction
   - Wallet prompts for signature
   ↓
5. Transaction Submission
   - Smart contract interaction
   - Compounds rewards
   ↓
6. Transaction Confirmation
   - Polls blockchain for confirmation
   - Updates UI
   ↓
7. Refresh Data
   - GET /stakingv2 (updates total staked)
   - Fetch user position (updates balance)
```

### Booster Activation Flow (Inferred)

```
1. User Clicks on Booster
   ↓
2. Frontend Checks User's Boosters
   - Blockchain RPC calls (via Web3 provider)
   - Gets user's NFT/booster inventory
   ↓
3. User Selects Booster to Activate
   ↓
4. Frontend Prepares Activation Transaction
   - Validates booster ownership
   - Calculates XP boost
   ↓
5. User Approves Activation Transaction
   - Wallet prompts for signature
   ↓
6. Transaction Submission
   - Smart contract interaction
   - Activates booster (5-day duration)
   ↓
7. Transaction Confirmation
   - Polls blockchain for confirmation
   - Updates UI (shows XP boost)
   ↓
8. Refresh Data
   - Fetch user position (updates XP APR)
```

### Quest Completion Flow (Inferred)

```
1. User Views Quest Details
   ↓
2. User Completes Tasks
   - Tasks may require actions on other platforms
   - (e.g., deposit on WOOFi Pro, trade on WOOFi Pro)
   ↓
3. Frontend Checks Task Completion
   - May query blockchain for transaction history
   - May query backend for verified completions
   ↓
4. User Claims Quest Rewards
   - If quest is completed and verified
   ↓
5. Frontend Prepares Claim Transaction
   - Validates quest completion
   - Calculates rewards (NFTs, tokens)
   ↓
6. User Approves Claim Transaction
   - Wallet prompts for signature
   ↓
7. Transaction Submission
   - Smart contract interaction
   - Claims quest rewards
   ↓
8. Transaction Confirmation
   - Polls blockchain for confirmation
   - Updates UI (shows claimed rewards)
   ↓
9. Refresh Data
   - GET /nft/campaign_infos (updates quest status)
   - Fetch user position (updates booster inventory)
```

## Network Request Patterns

### Request Headers

**GET Requests**:
```
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Cache-Control: no-cache
Pragma: no-cache
Referer: https://woofi.com/swap/stake
```

**POST Requests**:
```
Content-Type: application/json
Accept: application/json
```

### Response Headers

**API Responses**:
```
Content-Type: application/json
Access-Control-Allow-Origin: *
Cache-Control: public, max-age=...
```

## Query Parameters Reference

### Staking Endpoints

**Staking Info** (`/stakingv2`):
- No query parameters (returns global statistics)

**Reward Distribution** (`/reward_distribution`):
- No query parameters (returns weekly statistics)

**Surge Info** (`/surge/infos`):
- No query parameters (returns all surge campaigns)

**NFT Campaign Info** (`/nft/campaign_infos`):
- No query parameters (returns all quest campaigns)

**User-Specific Endpoints** (Likely):
- `/stakingv2?user_address={address}` - User's staking position (if exists)
- `/nft/campaign_infos?user_address={address}` - User's quest progress (if exists)

## Error Handling

**API Error Responses**:
```json
{
  "status": "error",
  "message": "Error message",
  "code": "ERROR_CODE"
}
```

**Success Response**:
```json
{
  "status": "ok",
  "data": { /* response data */ }
}
```

## Authentication

**Public Endpoints** (No Authentication):
- `/stakingv2`
- `/reward_distribution`
- `/surge/infos`
- `/nft/campaign_infos`

**User-Specific Data**:
- User staking positions: Fetched via blockchain RPC calls (requires wallet connection)
- User quest progress: May require wallet address (no API key needed)
- User booster inventory: Fetched via blockchain RPC calls

**Third-Party APIs**:
- Binance API: Public, no authentication required
- CoinGecko (via proxy): Public, no authentication required
- 1inch (via proxy): Public, no authentication required

## Caching Strategy

**Cached Resources**:
- NFT images/videos: Cached by browser/CDN
- Campaign images: Cached with long TTL
- Static assets: Cached with long TTL

**Non-Cached Resources**:
- Staking statistics: Fetched fresh (real-time data)
- Reward distribution: Updated periodically
- Campaign data: Updated periodically
- Prices: Updated frequently

## Rate Limiting

**Observation**:
- Multiple parallel requests on page load
- Periodic polling for staking statistics
- Frequent price updates
- No apparent rate limiting on public endpoints
- Third-party APIs may have rate limits

## Request Timing

**Initial Load**:
- Staking info: ~330ms
- Reward distribution: ~330ms
- Surge info: ~479ms
- NFT campaign info: ~153ms
- Token lists: ~473-2151ms (varies by network)
- Market data: ~377-451ms per page

**Total Initial Load**:
- Approximately 20-30 API requests
- Total load time: ~1-3 seconds (depending on network speed)

## Example: Complete Data Fetching Sequence

```
Page Load (t=0ms)
├── GET /stakingv2 (330ms)
├── GET /reward_distribution (330ms)
├── GET /surge/infos (479ms)
├── GET /nft/campaign_infos (153ms)
├── GET /okx_tokens (473ms)
├── GET /1inch_tokens?network=ethereum (2151ms)
├── GET /1inch_tokens?network=arbitrum (630ms)
├── GET /1inch_tokens?network=base (666ms)
├── GET /1inch_tokens?network=avax (580ms)
├── GET /1inch_tokens?network=zksync (649ms)
├── GET /1inch_tokens?network=optimism (576ms)
├── GET /1inch_tokens?network=polygon (666ms)
├── GET /1inch_tokens?network=bsc (678ms)
├── GET /coingecko/coins_markets?page=1 (451ms)
├── GET /coingecko/coins_markets?page=2 (377ms)
├── GET /coingecko/coins_markets?page=3 (377ms)
└── GET /api.binance.com/api/v3/ticker/price?symbol=WOOUSDT (87ms)

After Initial Load (t=3000ms+)
├── Periodic staking statistics updates
├── Price updates
└── User-specific data (when wallet connected)
```

## Key Features

### Staking System

- **Multi-chain Staking**: WOO tokens can be staked on multiple networks
- **Real Yield**: 80% revenue share from platform fees
- **XP System**: Earn XP for staking, boosts APR
- **Booster NFTs**: Boost XP APR for 5 days
- **Compounding**: Compound USDC rewards automatically

### Quest System

- **Mystery Box Quests**: Complete tasks to earn NFT rewards
- **Task Dependencies**: Some tasks unlock bonus tasks
- **Token Rewards**: Some quests offer token rewards
- **Multi-tier NFTs**: Uncommon, Rare, Epic tiers

### Surge Campaigns

- **Seasonal Events**: Time-limited staking campaigns
- **Multiplier Tasks**: Complete tasks for score multipliers
- **Token Rewards**: Earn tokens based on staking increase
- **Claim Periods**: Separate claim periods after campaign ends

### Reward System

- **USDC Rewards**: Earn USDC from platform revenue
- **XP Rewards**: Earn XP for staking
- **Weekly Distribution**: Rewards distributed weekly
- **Claim or Compound**: Users can claim or compound rewards

## Summary

WooFi's stake page uses:

1. **WooFi API** (`api.woofi.com`) for:
   - Staking statistics (`/stakingv2`)
   - Reward distribution (`/reward_distribution`)
   - Surge campaigns (`/surge/infos`)
   - Quest campaigns (`/nft/campaign_infos`)

2. **Third-Party APIs** for:
   - Price data (Binance)
   - Token metadata (1inch via proxy)
   - Market data (CoinGecko via proxy)

3. **Data Flow**:
   - **GET requests**: Fetch staking statistics, campaigns, rewards, token lists, market data
   - **POST requests**: Analytics tracking, blockchain transactions (via Web3 wallets)
   - **Real-time updates**: Periodic polling for staking statistics and rewards
   - **User interactions**: Stake/unstake/claim/compound handled via blockchain transactions

The page primarily **fetches data** (GET) for display purposes. POST requests are mainly used for analytics tracking. Actual staking operations (stake, unstake, claim, compound, booster activation) are handled through blockchain transactions via wallet connections, which don't appear in the HTTP network requests but are handled by Web3 providers.

