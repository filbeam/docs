# Payment Model

This document explains how FilBeam's pay-per-byte payment model works, including the complete flow from on-chain top-up to settlement.

## Overview

FilBeam uses a **hybrid on-chain/off-chain model**:

- **On-chain**: Top-ups, usage reporting, and settlement
- **Off-chain**: Quota calculation, caching, request serving, and usage tracking

This design provides usage and payment transparency while maintaining high performance for content delivery.

## Payment Rails Setup

When a data set is created with FilBeam enabled, the FWSS contract creates three payment rails:

```mermaid
sequenceDiagram
    participant User
    participant SP as Storage Provider
    participant PDPVerifier
    participant FWSS as FWSS Contract
    participant FP as Filecoin Pay

    User->>SP: Create data set with FilBeam enabled
    SP->>PDPVerifier: Create new data set
    PDPVerifier->>FWSS: Data set created
    FWSS->>FP: Create storage payment rail
    FWSS->>FP: Create CDN egress payment rail
    FWSS->>FP: Create cache-miss egress payment rail
    FWSS->>FWSS: Emit DataSetCreated event
```

| Payment Rail | Payee | Purpose |
|--------------|-------|---------|
| Storage | Storage Provider | Ongoing storage costs |
| CDN Egress | FilBeam | CDN delivery fees |
| Cache-Miss Egress | Storage Provider | Retrieval from origin |

## Complete Payment Flow

```mermaid
sequenceDiagram
    participant User
    participant FWSS as FWSS Contract<br/>(On-Chain)
    participant Indexer as FilBeam Indexer<br/>(Off-Chain)
    participant DB as FilBeam Database<br/>(Off-Chain)
    participant CDN as FilBeam CDN<br/>(Off-Chain)
    participant Reporter as Usage Reporter<br/>(Off-Chain)
    participant Operator as FilBeamOperator<br/>(On-Chain)
    participant Settler as Payment Settler<br/>(Off-Chain)
    participant SP as Storage Provider

    Note over User,SP: Phase 1: Top-Up (On-Chain)
    User->>FWSS: topUpCDNPaymentRails($10 CDN, $10 cache-miss)
    FWSS->>FWSS: Lock USDFC in payment rails
    FWSS-->>Indexer: Emit CdnPaymentRailsToppedUp event

    Note over User,SP: Phase 2: Quota Calculation (Off-Chain)
    Indexer->>DB: Calculate quota from USDFC amount
    Note right of DB: CDN: $7 → 1 TiB<br/>Cache-miss: $7 → 1 TiB
    Indexer->>DB: Credit quota to data set

    Note over User,SP: Phase 3: Content Delivery (Off-Chain)
    User->>CDN: Request content
    CDN->>DB: Check quota available
    DB-->>CDN: Quota OK
    CDN->>SP: Fetch content (cache miss)
    SP-->>CDN: Return content
    CDN->>DB: Log retrieval (bytes, cache-miss flag)
    CDN->>DB: Decrement quotas
    CDN-->>User: Serve content

    Note over User,SP: Phase 4: Usage Reporting (On-Chain)
    Reporter->>DB: Aggregate retrieval logs
    Reporter->>Operator: recordUsageRollups(dataSetIds, cdnBytes, cacheMissBytes)
    Operator->>Operator: Calculate & store settlement amounts

    Note over User,SP: Phase 5: Settlement (On-Chain)
    Settler->>Operator: settleCDNPaymentRails(dataSetIds)
    Operator->>FWSS: Settle with pre-calculated amount
    FWSS->>FWSS: Transfer CDN fees to FilBeam
    SP->>Operator: settleCacheMissPaymentRails(dataSetIds)
    Operator->>FWSS: Settle with pre-calculated amount
    FWSS->>FWSS: Transfer cache-miss fees ($7/TiB)
```

## Phase 1: Top-Up (On-Chain)

The user tops up their FilBeam payment rails by calling the FWSS contract:

```javascript
import { createPublicClient, createWalletClient, http, parseUnits } from 'viem'
import * as synapseCore from '@filoz/synapse-core'

const chain = synapseCore.chains.calibration
const storageContract = chain.contracts.storage

// Top up with $10 for CDN and $10 for cache-miss
const cdnAmount = parseUnits('10', 18)      // USDFC has 18 decimals
const cacheMissAmount = parseUnits('10', 18)

await walletClient.writeContract({
  address: storageContract.address,
  abi: storageContract.abi,
  functionName: 'topUpCDNPaymentRails',
  args: [dataSetId, cdnAmount, cacheMissAmount]
})
```

**What happens on-chain:**
1. USDFC is locked in payment rails (CDN rail + cache-miss rail)
2. Contract emits `CDNPaymentRailsToppedUp` event with amounts
3. Funds are reserved but not yet transferred to anyone

## Phase 2: Quota Calculation (Off-Chain)

The FilBeam Indexer receives the blockchain event and calculates quotas:

```
CDN Quota (bytes) = (USDFC amount × BYTES_PER_TIB) / CDN_RATE
Cache Miss Quota (bytes) = (USDFC amount × BYTES_PER_TIB) / CACHE_MISS_RATE
```

Where:
- `BYTES_PER_TIB` = 1,099,511,627,776 (1024^4)
- `CDN_RATE` = 7 × 10^18 (with 18 decimals = $7)
- `CACHE_MISS_RATE` = 7 × 10^18 (with 18 decimals = $7)

**Both rails have the same $7/TiB rate.** The $14/TiB effective cost for cache misses comes from being charged on BOTH rails simultaneously.

**Example: $7 CDN + $7 cache-miss top-up**
```
CDN Quota = (7 × 10^18 × 1,099,511,627,776) / (7 × 10^18)
          = 1,099,511,627,776 bytes
          = 1 TiB

Cache Miss Quota = (7 × 10^18 × 1,099,511,627,776) / (7 × 10^18)
                 = 1,099,511,627,776 bytes
                 = 1 TiB
```

The updated quota is stored in FilBeam's D1 database.

## Phase 3: Content Delivery (Off-Chain)

When users request content, FilBeam serves it and tracks usage:

```mermaid
sequenceDiagram
    participant User
    participant CDN as FilBeam CDN
    participant Cache
    participant DB as Database
    participant SP as Storage Provider

    User->>CDN: GET /piece/{cid}
    CDN->>Cache: Check cache
    alt Cache Hit
        Cache-->>CDN: Return content
        CDN-->>User: Serve content
        CDN->>DB: Log (egress bytes, cache_miss=false)
        DB->>DB: Decrement CDN quota only
    else Cache Miss
        Cache-->>CDN: Not found
        CDN->>DB: Check quotas
        CDN->>SP: Fetch content
        SP-->>CDN: Return content
        CDN->>Cache: Store content
        CDN-->>User: Serve content
        CDN->>DB: Log (egress bytes, cache_miss=true)
        DB->>DB: Decrement BOTH quotas
    end
```

**Quota deduction rules:**

| Request Type | CDN Quota | Cache Miss Quota |
|--------------|-----------|------------------|
| Cache Hit | -N bytes | unchanged |
| Cache Miss | -N bytes | -N bytes |

Each request is logged to D1 for future processing.

## Phase 4: Usage Reporting (On-Chain)

Periodically, the Usage Reporter aggregates logs and reports to the blockchain.

**On-chain reporting:**
```javascript
await walletClient.writeContract({
  address: filBeamOperator,
  abi: filBeamOperatorAbi,
  functionName: 'recordUsageRollups',
  args: [
    toEpoch,           // Filecoin epoch up to which usage is reported
    dataSetIds,        // Array of data set IDs
    cdnBytesUsed,      // Array of CDN bytes per data set
    cacheMissBytesUsed // Array of cache-miss bytes per data set
  ]
})
```

**Reporting schedule:**
- Calibration testnet: Every 30 minutes
- Mainnet: Every 4 hours

## Phase 5: Settlement (On-Chain)

Settlement transfers funds from payment rails to **recipient's Filecoin Pay account**. Both FilBeam and storage providers call the **FilBeamOperator** contract to settle their respective rails.

### How Settlement Amounts Are Calculated

Settlement amounts are **pre-calculated during usage reporting** (Phase 4), not during settlement:

```
CDN Amount = reportedCdnBytes × cdnRatePerByte
Cache-Miss Amount = reportedCacheMissBytes × cacheMissRatePerByte
```

These amounts are stored in the contract and accumulate with each usage report. Settlement simply transfers the accumulated amounts.

### CDN Settlement (FilBeam)

FilBeam calls the FilBeamOperator to settle the CDN rail:

```javascript
await walletClient.writeContract({
  address: filBeamOperator,
  abi: filBeamOperatorAbi,
  functionName: 'settleCDNPaymentRails',
  args: [dataSetIds]
})
```

### Cache-Miss Settlement (Storage Providers)

Storage providers call the FilBeamOperator to settle the cache-miss rail:

```javascript
await walletClient.writeContract({
  address: filBeamOperator,
  abi: filBeamOperatorAbi,
  functionName: 'settleCacheMissPaymentRails',
  args: [dataSetIds]
})
```

### Settlement Flow

```mermaid
sequenceDiagram
    participant FB as FilBeam
    participant SP as Storage Provider
    participant FBA as FilBeam Filecoin Pay Account
    participant SPA as SP Filecoin Pay Account
    participant Operator as FilBeamOperator
    participant FWSS as FWSS Contract
    participant Rail as Payment Rail<br/>(Locked USDFC)

    Note over FB,Rail: Amounts pre-calculated during usage reporting
    Note over Operator: Stored: cdnAmount, cacheMissAmount

    Note over FB,Rail: CDN Settlement
    FB->>Operator: settleCDNPaymentRails([dataSetIds])
    Operator->>Operator: Get stored cdnAmount
    Operator->>FWSS: Settle rail with cdnAmount
    FWSS->>Rail: Release funds
    Rail->>FBA: Transfer USDFC

    Note over FB,Rail: Cache-Miss Settlement
    SP->>Operator: settleCacheMissPaymentRails([dataSetIds])
    Operator->>Operator: Get stored cacheMissAmount
    Operator->>FWSS: Settle rail with cacheMissAmount
    FWSS->>Rail: Release funds
    Rail->>SPA: Transfer USDFC
```

### Settlement Details

- **Anyone can call** the settlement methods, but funds go to the designated payee's Filecoin Pay account
- **Partial settlements** are supported if locked funds are insufficient
- **Amounts accumulate** between settlements - no need to settle after every usage report

## Cost Breakdown

Both payment rails charge **$7/TiB**. The effective cost depends on request type:

| Request Type | CDN Rail | Cache-Miss Rail | Total Cost |
|--------------|----------|-----------------|------------|
| **Cache Hit** | $7/TiB | — | **$7/TiB** |
| **Cache Miss** | $7/TiB | $7/TiB | **$14/TiB** |

The $14/TiB cache-miss cost comes from being charged on BOTH rails:

| Recipient | Amount | Settlement Method |
|-----------|--------|-------------------|
| **FilBeam CDN** | $7/TiB | `settleCDNPaymentRails()` |
| **Storage Provider** | $7/TiB | `settleCacheMissPaymentRails()` |

## On-Chain vs Off-Chain Summary

| Component | Location | Purpose |
|-----------|----------|---------|
| Payment rails | On-chain | Lock funds, guarantee payment |
| Top-up events | On-chain | Trigger quota calculation |
| Quota tracking | Off-chain | Fast request validation |
| Request serving | Off-chain | Low-latency content delivery |
| Usage logging | Off-chain | Track egress for billing |
| Usage reporting | On-chain | Transparent usage records |
| Settlement | On-chain | Transfer funds to recipients |

## Why This Model?

### For Users
- **Predictable costs**: Know exactly what you're paying upfront
- **No surprise bills**: Service stops when quota exhausted
- **Price protection**: Purchased quota keeps its rate
- **Transparent**: All payments and usage on-chain

### For Storage Providers
- **Guaranteed payment**: Funds locked before service
- **Self-service settlement**: Claim earnings anytime
- **Fair compensation**: $7/TiB for actual bytes delivered

### For the Network
- **Decentralized**: No central billing authority
- **Transparent**: All transactions queriable on-chain
- **Trustless**: Smart contracts enforce payment rules

## Monitoring Your Account

### Check Quota

```javascript
const STATS_API = 'https://calibration.stats.filbeam.io'

const response = await fetch(`${STATS_API}/data-set/${dataSetId}`)
const stats = await response.json()

const cdnQuotaGiB = Number(stats.cdnEgressQuota) / (1024 ** 3)
console.log('CDN Quota:', cdnQuotaGiB.toFixed(2), 'GiB')
```

## Service Termination

Currently FilBeam service can only be terminated by terminating the full service (deleting the data set):

```mermaid
sequenceDiagram
    participant User
    participant FWSS as FWSS Contract
    participant FP as Filecoin Pay

    User->>FWSS: Terminate service
    FWSS->>FP: Terminate storage payment rail
    FWSS->>FP: Terminate CDN egress payment rail
    FWSS->>FP: Terminate cache-miss egress payment rail
    FWSS->>FWSS: Emit CdnServiceTerminated event
    FWSS->>FWSS: Emit ServiceTerminated event
```

## See Also

- [Quota System](quota-system.md) - Understanding dual quotas
- [Usage Reporting](usage-reporting.md) - How usage is reported on-chain
- [Pricing Reference](../pricing.md) - Detailed pricing information
- [Top Up CDN Quota](../how-to/top-up-cdn-quota.md) - Step-by-step guide
