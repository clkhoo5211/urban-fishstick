# Dune Analytics WooFi Dashboard - Network Fetching & Posting Analysis

This document analyzes how the [Dune Analytics WooFi dashboard](https://dune.com/woofianalytics/woofi-dashboard) fetches and posts data, including all API endpoints, request/response structures, and data flow patterns.

## Overview

The Dune Analytics WooFi dashboard is a comprehensive analytics dashboard hosted on Dune Analytics that displays WooFi Swap metrics, trading volumes, fees, user statistics, and chain breakdowns. The dashboard uses Dune's GraphQL API and execution API to fetch query results and render visualizations.

## Key API Endpoints

### Base API URLs
- **Dune GraphQL API**: `https://dune.com/public/graphql`
- **Dune Execution API**: `https://core-api.dune.com/public/execution`
- **Dune Public Events API**: `https://core-api.dune.com/public/events/page-view/`
- **Dune Assets**: `https://dune.com/assets/`
- **Dune Media**: `https://prod-dune-media.s3.eu-west-1.amazonaws.com/`

## Network Requests Breakdown

### 1. Initial Page Load Requests

#### GET Requests (Data Fetching)

**1. Dashboard Metadata (GraphQL)**
```
POST https://dune.com/public/graphql
```
- **Purpose**: Fetch dashboard metadata, user information, and query list
- **Timing**: Initial page load
- **Method**: POST (GraphQL)
- **Count**: Multiple requests (17 observed)
- **Response**: Dashboard configuration, query IDs, visualization settings

**2. Query Execution (Execution API)**
```
POST https://core-api.dune.com/public/execution
```
- **Purpose**: Execute SQL queries and fetch results
- **Timing**: Initial page load, periodic refreshes
- **Method**: POST
- **Count**: Multiple requests (30 observed, one per visualization)
- **Response**: Query results as JSON data

**3. Page View Tracking**
```
POST https://core-api.dune.com/public/events/page-view/
```
- **Purpose**: Track page views for analytics
- **Timing**: Initial page load
- **Method**: POST

**4. User Profile Data**
```
GET https://dune.com/_next/data/{buildId}/woofianalytics.json?slug=woofianalytics
```
- **Purpose**: Fetch user/profile metadata
- **Timing**: Initial page load
- **Method**: GET

**5. Color Palettes**
```
GET https://dune.com/api/color-palettes/
```
- **Purpose**: Fetch color palette configurations for charts
- **Timing**: Initial page load
- **Method**: GET

**6. Feature Flags**
```
GET https://flag.lab.productmetrics.dune.com/sdk/v2/flags
GET https://api.lab.productmetrics.dune.com/sdk/v2/vardata?v=0
```
- **Purpose**: Fetch feature flags and A/B testing configurations
- **Timing**: Initial page load
- **Method**: GET

#### POST Requests

**1. GraphQL Queries**
```
POST https://dune.com/public/graphql
```
- **Purpose**: GraphQL queries for dashboard metadata, query definitions, visualization settings
- **Timing**: Initial page load, user interactions
- **Method**: POST
- **Content-Type**: application/json
- **Payload**: GraphQL query with variables

**2. Query Execution**
```
POST https://core-api.dune.com/public/execution
```
- **Purpose**: Execute SQL queries and return results
- **Timing**: Initial page load, periodic refreshes (3h, 4h intervals)
- **Method**: POST
- **Content-Type**: application/json
- **Payload**: Query execution request with query ID and parameters

**3. Analytics Tracking**
```
POST https://www.google-analytics.com/g/collect
POST https://o507319.ingest.sentry.io/api/6138642/envelope/
POST https://cca-lite.coinbase.com/metrics
```
- **Purpose**: User analytics and error tracking
- **Timing**: On page load and user interactions
- **Method**: POST

## Dashboard Components

Based on the page snapshot, the dashboard contains the following visualizations:

### 1. Dashboard Controls
- **Chain Filter**: Filter by blockchain (All, Arbitrum, Base, etc.)
- **Interval Filter**: Filter by time interval (day, week, month)

### 2. Visualizations

**1. WOOFi EVM Ranking on Defillama**
- Query ID: 5349448 / 8762237
- Status: "No results from query"

**2. WOOFi Swap EVM Ranking on Defillama**
- Query ID: 5349217 / 8761980
- Status: "No results from query"

**3. WOOFi Swap Ranking on Defillama**
- Query ID: 4882020 / 8086432
- Status: "No results from query"

**4. BTC Daily Swap Volume**
- Query ID: 5587449 / 9091899
- Type: Column chart
- Data: Daily BTC swap volume on Arbitrum
- Refresh: 3h

**5. Daily Cross Swap Volume on Destination Chain**
- Query ID: 4860259 / 8049442
- Type: Column chart
- Data: Cross-chain swap volume by destination chain
- Chains: All, linea, mantle, ethereum, optimism, avalanche_c, bnb, base, arbitrum, polygon
- Refresh: 3h

**6. Daily Swap Top 10 Pair Breakdown**
- Query ID: 4836530 / 8013290
- Type: Column chart
- Data: Top 10 trading pairs by volume
- Pairs: USDT0-WETH, USDC-WETH.e, USDt-WAVAX, USDT-BTCB, WETH.e-USDC, etc.
- Refresh: 3h

**7. EVM Daily Swap Volume (Chain Breakdown)**
- Query ID: 4836566 / 8013425
- Type: Column chart
- Data: Daily swap volume by EVM chain
- Chains: All, bnb, sonic, optimism, avalanche_c, linea, arbitrum, base, mantle, berachain, polygon, hyperevm
- Refresh: 3h

**8. Last 24hr Volume Rolling**
- Query ID: 4837987 / 8016080
- Type: Number display
- Value: $28,525,588
- Refresh: 3h

**9. 24hr Fee**
- Query ID: 4838011 / 8016095
- Type: Number display
- Value: $5,204
- Refresh: 3h

**10. Daily Swap Source Breakdown**
- Query ID: 4836687 / 8013734
- Type: Column chart
- Data: Trading volume by source aggregator
- Sources: All, velora, woofi, 0x, openocean, yieldyak, 1inch, odos, kyberswap
- Refresh: 3h

**11. 7-Day Fee**
- Query ID: 4848414 / 8032063
- Type: Number display
- Value: $29,895.56 USDT
- Refresh: 3h

**12. Daily Users Chain Breakdown**
- Query ID: 5058022 / 8352134
- Type: Column chart
- Data: Daily unique users by chain
- Chains: All, polygon, base, optimism, arbitrum, mantle, linea, bnb, avalanche_c, sonic, berachain, hyperevm
- Refresh: 3h

**13. Daily Trades Chain Breakdown**
- Query ID: 5058094 / 8352209
- Type: Column chart
- Data: Daily trades by chain
- Chains: All, bnb, hyperevm, berachain, mantle, linea, polygon, avalanche_c, optimism, arbitrum, sonic, base
- Refresh: 3h

**14. Solana Daily Swap Volume**
- Query ID: 5126595 / 8452526
- Type: Column chart
- Data: Daily swap volume on Solana
- Refresh: 4h

**15. Daily Cross Swap Volume on Source Chain**
- Query ID: 4858987 / 8049363
- Type: Column chart
- Data: Cross-chain swap volume by source chain
- Chains: All, ethereum, avalanche_c, polygon, linea, optimism, mantle, arbitrum, base, bnb
- Refresh: 3h

## API Request Patterns

### GraphQL Request Structure

**Endpoint**: `POST https://dune.com/public/graphql`

**Request Headers**:
```
Content-Type: application/json
Accept: application/json
```

**Request Body** (Inferred):
```json
{
  "query": "...",
  "variables": {
    "slug": "woofianalytics",
    "dashboardSlug": "woofi-dashboard"
  }
}
```

**Example GraphQL Queries** (Based on dashboard functionality):
- Dashboard metadata query
- Query definitions query
- Visualization settings query
- User profile query

### Execution API Request Structure

**Endpoint**: `POST https://core-api.dune.com/public/execution`

**Request Headers**:
```
Content-Type: application/json
Accept: application/json
```

**Request Body** (Inferred):
```json
{
  "query_id": 4836530,
  "parameters": {
    "chain": "All",
    "interval": "day"
  }
}
```

**Response Structure** (Inferred):
```json
{
  "execution_id": "...",
  "state": "COMPLETED",
  "result": {
    "rows": [
      {
        "date": "2025-10-03",
        "volume": 1000000,
        "chain": "arbitrum"
      }
    ],
    "metadata": {
      "column_names": ["date", "volume", "chain"],
      "result_set_bytes": 1024,
      "total_row_count": 100
    }
  }
}
```

## Data Fetching Flow

### Initial Page Load Sequence

```
1. Page HTML Loads
   ↓
2. JavaScript Bundles Load (Next.js)
   ↓
3. GraphQL Queries (Parallel):
   - POST /public/graphql (dashboard metadata)
   - POST /public/graphql (query definitions)
   - POST /public/graphql (visualization settings)
   - POST /public/graphql (user profile)
   ↓
4. Query Executions (Parallel):
   - POST /core-api.dune.com/public/execution (for each visualization)
   - 30+ execution requests (one per chart/visualization)
   ↓
5. Configuration Fetching:
   - GET /api/color-palettes/
   - GET /flag.lab.productmetrics.dune.com/sdk/v2/flags
   - GET /api.lab.productmetrics.dune.com/sdk/v2/vardata
   ↓
6. Analytics Tracking:
   - POST /core-api.dune.com/public/events/page-view/
   - POST /www.google-analytics.com/g/collect
```

### Real-Time Updates

**Polling Pattern**:
- Visualizations refresh automatically:
  - Most charts: Every 3 hours (3h)
  - Solana chart: Every 4 hours (4h)
- User interactions (filter changes) trigger new executions
- Refresh button manually triggers new executions

**Filter Changes**:
- Changing chain filter triggers new execution requests
- Changing interval filter triggers new execution requests
- Each visualization executes its query independently

## Query Parameter System

### Dashboard Parameters

**Chain Filter**:
- `All`: All chains
- `arbitrum`: Arbitrum
- `base`: Base
- `bnb`: BNB Chain
- `polygon`: Polygon
- `optimism`: Optimism
- `avalanche_c`: Avalanche
- `mantle`: Mantle
- `linea`: Linea
- `sonic`: Sonic
- `berachain`: Berachain
- `hyperevm`: HyperEVM
- `ethereum`: Ethereum

**Interval Filter**:
- `day`: Daily
- `week`: Weekly
- `month`: Monthly

### Query-Specific Parameters

Each query may have its own parameters:
- Date ranges
- Token filters
- Chain filters
- Aggregation levels

## Authentication

**Public Endpoints** (No Authentication):
- `/public/graphql`: Public GraphQL endpoint (read-only)
- `/core-api.dune.com/public/execution`: Public execution endpoint (read-only)
- Dashboard viewing: Public access (no login required)

**Authentication Required**:
- Dashboard editing: Requires login
- Query editing: Requires login
- Favoriting: Requires login (`/auth/login?next=...`)
- Screenshot: Requires login

**Authentication Flow**:
- Clicking "Favorite" or "Screenshot" redirects to: `/auth/login?next=%2Fwoofianalytics%2Fwoofi-dashboard`
- Uses OAuth/login system
- Session management via cookies

## Error Handling

**Query Execution Errors**:
- "No results from query": Query returned no data or failed
- Execution may be queued or in progress
- Errors are displayed in the UI

**GraphQL Errors**:
- Standard GraphQL error format
- Errors shown in UI or console

## Caching Strategy

**Cached Resources**:
- JavaScript bundles: Cached by Next.js build ID
- CSS files: Cached with build ID
- Images: Cached by browser/CDN
- Color palettes: Cached

**Non-Cached Resources**:
- Query results: Fresh execution each time
- GraphQL responses: Fresh each request
- Dashboard metadata: May be cached briefly

## Rate Limiting

**Observation**:
- 30+ parallel execution requests on page load
- Multiple GraphQL requests in parallel
- No apparent rate limiting on public endpoints
- Execution requests may be queued server-side

## Request Timing

**Initial Load**:
- GraphQL queries: ~515-841ms each
- Execution requests: ~519-2432ms each (varies by query complexity)
- Total initial load: ~10-15 seconds (with all visualizations)

**Refresh Intervals**:
- Most visualizations: 3 hours
- Solana chart: 4 hours
- Manual refresh: Immediate

## Example: Complete Data Fetching Sequence

```
Page Load (t=0ms)
├── GraphQL Queries (Parallel):
│   ├── POST /public/graphql (dashboard metadata) (568ms)
│   ├── POST /public/graphql (query definitions) (515ms)
│   ├── POST /public/graphql (visualization settings) (515ms)
│   └── ... (17 total GraphQL requests)
│
├── Query Executions (Parallel):
│   ├── POST /core-api.dune.com/public/execution (query 4836530) (2080ms)
│   ├── POST /core-api.dune.com/public/execution (query 4836566) (2038ms)
│   ├── POST /core-api.dune.com/public/execution (query 4837987) (2432ms)
│   ├── POST /core-api.dune.com/public/execution (query 4838011) (1997ms)
│   ├── POST /core-api.dune.com/public/execution (query 4836687) (2021ms)
│   ├── POST /core-api.dune.com/public/execution (query 4848414) (1707ms)
│   ├── POST /core-api.dune.com/public/execution (query 5058022) (2162ms)
│   ├── POST /core-api.dune.com/public/execution (query 5058094) (2184ms)
│   ├── POST /core-api.dune.com/public/execution (query 5126595) (2112ms)
│   ├── POST /core-api.dune.com/public/execution (query 4860259) (2102ms)
│   ├── POST /core-api.dune.com/public/execution (query 5587449) (2136ms)
│   ├── POST /core-api.dune.com/public/execution (query 4858987) (2143ms)
│   └── ... (30 total execution requests)
│
├── Configuration (Parallel):
│   ├── GET /api/color-palettes/ (79ms)
│   ├── GET /flag.lab.productmetrics.dune.com/sdk/v2/flags (568ms)
│   └── GET /api.lab.productmetrics.dune.com/sdk/v2/vardata (660ms)
│
└── Analytics:
    ├── POST /core-api.dune.com/public/events/page-view/ (2594ms)
    └── POST /www.google-analytics.com/g/collect (20ms)

After Initial Load (t=15000ms+)
├── Automatic refresh every 3-4 hours
├── Manual refresh on button click
└── Filter changes trigger new executions
```

## Key Features

### Dashboard Functionality

- **Multi-Chain Analytics**: Supports 10+ EVM chains plus Solana
- **Real-Time Updates**: Automatic refresh every 3-4 hours
- **Interactive Filters**: Chain and interval filters
- **Multiple Visualizations**: 15+ charts and metrics
- **Query Transparency**: Each visualization links to its source query

### Data Sources

- **On-Chain Data**: Query results from blockchain data indexed by Dune
- **SQL Queries**: Each visualization is backed by a SQL query
- **Aggregated Metrics**: Pre-computed and real-time aggregations
- **Cross-Chain Analysis**: Unified view across multiple chains

### Visualization Types

- **Column Charts**: Time-series volume data
- **Number Displays**: Single metric values (24h volume, fees)
- **Multi-Series Charts**: Breakdown by chain, pair, source
- **Interactive Legends**: Click to filter series
- **Time Range Selectors**: X-axis date ranges

## Integration with WooFi

### Redirect from WooFi Dashboard

The Dune dashboard is referenced from WooFi's official dashboard:
- Link: "View more on Dune" (visible in WooFi dashboard)
- Direct link: `https://dune.com/woofianalytics/woofi-dashboard`
- Purpose: Provides more detailed analytics than WooFi's native dashboard

### Dashboard Links

The Dune dashboard includes links back to WooFi:
- "Official Dashboard": `https://woofi.com/dashboard`
- "Swap Page": `https://woofi.com/#/`
- "WOO Stake": `https://dune.com/woofianalytics/woofi-staking`
- "WOO Buyback & Burn": `https://dune.com/woofianalytics/woo-buyback-and-burn`
- "Developer Docs": `https://learn.woo.org/dev-docs`

## Summary

The Dune Analytics WooFi dashboard uses:

1. **Dune GraphQL API** (`dune.com/public/graphql`) for:
   - Dashboard metadata
   - Query definitions
   - Visualization settings
   - User profile data

2. **Dune Execution API** (`core-api.dune.com/public/execution`) for:
   - Executing SQL queries
   - Fetching query results
   - Updating visualizations

3. **Data Flow**:
   - **POST requests**: GraphQL queries, query executions, analytics tracking
   - **GET requests**: Configuration, feature flags, color palettes
   - **Real-time updates**: Automatic refresh every 3-4 hours
   - **Interactive filters**: Chain and interval filters trigger new executions

The dashboard is a **read-only analytics platform** that fetches and displays blockchain analytics data. All data is fetched via POST requests (GraphQL and execution API), with GET requests used only for configuration and static assets. The dashboard provides comprehensive analytics across multiple chains and time periods, with automatic refresh intervals and interactive filtering capabilities.

## Technical Notes

### Dune Analytics Architecture

- **SQL Queries**: Each visualization is backed by a SQL query against indexed blockchain data
- **Query Execution**: Queries are executed on-demand via the execution API
- **Result Caching**: Results may be cached briefly on the server side
- **GraphQL Schema**: Dune uses GraphQL for metadata and configuration
- **Next.js Frontend**: Dashboard is built with Next.js (React framework)

### Performance Considerations

- **Parallel Execution**: All queries execute in parallel for faster loading
- **Query Complexity**: Execution time varies by query complexity (519ms - 2432ms)
- **Refresh Intervals**: 3-4 hour intervals balance freshness and server load
- **Client-Side Rendering**: Charts rendered client-side from query results

