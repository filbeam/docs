---
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# Top Up CDN Quota

This guide explains how to increase your FilBeam CDN egress quota when it's running low.

## When to Top Up

Top up your quota when:

- Your CDN egress quota is below your expected usage
- You receive 402 errors ("CDN egress quota exhausted")
- Your monitoring shows quota running low

## Prerequisites

- Node.js 18+
- Private key with USDFC balance deposited in Filecoin Pay
- Existing data set with CDN enabled

## Step 1: Check Current Quota

First, check your current quota levels via the Stats API:

```javascript
const STATS_API = 'https://calibration.stats.filbeam.io'
const dataSetId = 3830

const response = await fetch(`${STATS_API}/data-set/${dataSetId}`)
const stats = await response.json()

const TIB = 1024 ** 4
const cdnQuotaTiB = Number(stats.cdnEgressQuota) / TIB
const cacheMissQuotaTiB = Number(stats.cacheMissEgressQuota) / TIB

console.log('Current CDN Quota:', cdnQuotaTiB.toFixed(3), 'TiB')
console.log('Current Cache Miss Quota:', cacheMissQuotaTiB.toFixed(3), 'TiB')
```

## Step 2: Top Up Using Synapse SDK

Use the Synapse SDK with ethers.js:

```javascript
import { Synapse, WarmStorageService, RPC_URLS } from '@filoz/synapse-sdk'
import { ethers } from 'ethers'

// Initialize SDK
const synapse = await Synapse.create({
  privateKey: process.env.PRIVATE_KEY,
  rpcURL: RPC_URLS.calibration.http
})

// Initialize WarmStorageService
const warmStorage = await WarmStorageService.create(synapse.getProvider(), synapse.getWarmStorageAddress())

const dataSetId = 3830

// Top up: $7 USDFC for CDN, $7 USDFC for cache-miss (1 TiB each)
// USDFC has 18 decimals
const cdnAmount = ethers.parseUnits('7', 18)        // $7 = 1 TiB CDN quota
const cacheMissAmount = ethers.parseUnits('7', 18)  // $7 = 1 TiB cache-miss quota

console.log('Topping up CDN quota...')
const tx = await warmStorageService.topUpCDNPaymentRails(synapse.getSigner(), dataSetId, cdnAmount, cacheMissAmount)

console.log('Transaction hash:', tx.hash)
const receipt = await tx.wait()
console.log('Status:', receipt.status === 1 ? 'Confirmed!' : 'Failed')
```

## Step 3: Verify New Quota

Confirm the quota was added:

```javascript
// Wait a few seconds for indexing
await new Promise(r => setTimeout(r, 5000))

const response = await fetch(`${STATS_API}/data-set/${dataSetId}`)
const newStats = await response.json()

const TIB = 1024 ** 4
const newCdnQuotaTiB = Number(newStats.cdnEgressQuota) / TIB
const newCacheMissQuotaTiB = Number(newStats.cacheMissEgressQuota) / TIB

console.log('New CDN Quota:', newCdnQuotaTiB.toFixed(3), 'TiB')
console.log('New Cache Miss Quota:', newCacheMissQuotaTiB.toFixed(3), 'TiB')
```

## Quota Calculation

Both rails have the same $7/TiB rate:

```
CDN Quota (bytes) = (USDFC amount × BYTES_PER_TIB) / RATE_PER_TIB
Cache Miss Quota (bytes) = (USDFC amount × BYTES_PER_TIB) / RATE_PER_TIB
```

Where:
- `BYTES_PER_TIB` = 1,099,511,627,776 (1024^4)
- `RATE_PER_TIB` = $7 (7 × 10^18 in USDFC with 18 decimals)

### Example Calculation

For $7 USDFC to each rail:

```
CDN Quota = (7 × 10^18 × 1,099,511,627,776) / (7 × 10^18)
          = 1,099,511,627,776 bytes
          = 1 TiB

Cache Miss Quota = (7 × 10^18 × 1,099,511,627,776) / (7 × 10^18)
                 = 1,099,511,627,776 bytes
                 = 1 TiB
```

### Understanding Effective Costs

Both rails charge $7/TiB independently. The effective cost depends on cache behavior:

| Request Type | CDN Rail | Cache-Miss Rail | Effective Cost |
|--------------|----------|-----------------|----------------|
| Cache Hit | $7/TiB | — | **$7/TiB** |
| Cache Miss | $7/TiB | $7/TiB | **$14/TiB** |

**Important:** A cache miss consumes from BOTH quotas. To serve 1 TiB of cache-miss traffic, you need:
- 1 TiB CDN quota ($7)
- 1 TiB cache-miss quota ($7)
- Total: $14

## Quick Reference: Cost per TiB

| Rail | Rate |
|------|------|
| CDN | $7/TiB |
| Cache-Miss | $7/TiB |
| **Cache miss (both rails)** | $14/TiB |

## Top Up Helper Function

```javascript
import { ethers } from 'ethers'

/**
 * Calculate USDFC needed for desired quota on a single rail
 * Both rails have the same $7/TiB rate
 * @param {number} tebibytes - Desired quota in TiB
 * @returns {bigint} Amount in USDFC (18 decimals)
 */
function calculateTopUpAmount(tebibytes) {
  const ratePerTiB = 7n * (10n ** 18n)  // $7 in USDFC (18 decimals)
  // Convert TiB to bigint (multiply by 1000 to handle decimals, then divide)
  const tibWhole = BigInt(Math.floor(tebibytes * 1000))
  return (tibWhole * ratePerTiB) / 1000n
}

// Example: Top up to add 1 TiB of quota to each rail
const amount = calculateTopUpAmount(1)
console.log('Amount per rail (1 TiB):', ethers.formatUnits(amount, 18), 'USDFC')  // $7.00

// For 0.5 TiB cache-miss capacity (need both rails)
const cdnAmount = calculateTopUpAmount(0.5)       // $3.50
const cacheMissAmount = calculateTopUpAmount(0.5) // $3.50
console.log('Total for 0.5 TiB cache-miss:', ethers.formatUnits(cdnAmount + cacheMissAmount, 18), 'USDFC')  // $7.00
```

## Complete Top-Up Script

```javascript
import { Synapse, WarmStorageService, RPC_URLS } from '@filoz/synapse-sdk'
import { ethers } from 'ethers'

const STATS_API = 'https://calibration.stats.filbeam.io'

async function topUpCDNQuota(privateKey, dataSetId, cdnUsdfc, cacheMissUsdfc = 0) {
  // Initialize SDK
  const synapse = await Synapse.create({
    privateKey,
    rpcURL: RPC_URLS.calibration.http
  })

  // Initialize WarmStorageService
  const warmStorage = await WarmStorageService.create(synapse.getProvider(), synapse.getWarmStorageAddress())

  const address = await synapse.getSigner().getAddress()
  console.log('Connected as:', address)

  // Check current quota
  console.log('Checking current quota...')
  const beforeRes = await fetch(`${STATS_API}/data-set/${dataSetId}`)
  const beforeStats = await beforeRes.json()
  const TIB = 1024 ** 4
  console.log('Current CDN quota:', (Number(beforeStats.cdnEgressQuota) / TIB).toFixed(3), 'TiB')

  // Calculate amounts
  const cdnAmount = ethers.parseUnits(cdnUsdfc.toString(), 18)
  const cacheMissAmount = ethers.parseUnits(cacheMissUsdfc.toString(), 18)

  console.log(`Topping up with $${cdnUsdfc} CDN, $${cacheMissUsdfc} cache-miss...`)

  // Execute transaction
  const tx = await warmStorageService.topUpCDNPaymentRails(
    synapse.getSigner(), 
    dataSetId, 
    cdnAmount, 
    cacheMissAmount
  )

  console.log('Transaction:', tx.hash)
  const receipt = await tx.wait()
  console.log('Status:', receipt.status === 1 ? '✅ Confirmed' : '❌ Failed')

  // Verify new quota
  await new Promise(r => setTimeout(r, 3000))
  const afterRes = await fetch(`${STATS_API}/data-set/${dataSetId}`)
  const afterStats = await afterRes.json()
  console.log('New CDN quota:', (Number(afterStats.cdnEgressQuota) / TIB).toFixed(3), 'TiB')

  return {
    hash: tx.hash,
    status: receipt.status,
    beforeQuota: beforeStats.cdnEgressQuota,
    afterQuota: afterStats.cdnEgressQuota
  }
}

// Usage: Top up with $7 for 1 TiB CDN quota
await topUpCDNQuota(
  process.env.PRIVATE_KEY,
  3830,    // Data set ID
  7,       // $7 USDFC for CDN (1 TiB)
  7        // $7 USDFC for cache-miss (1 TiB)
)
```

## Troubleshooting

### "Insufficient funds" Error

Your Filecoin Pay account doesn't have enough USDFC. Deposit more funds first:

```javascript
const amount = ethers.parseUnits('20', 18)  // $20 USDFC
const tx = await synapse.payments.depositWithPermit(amount)
await tx.wait()
```

### "Invalid data set" Error

Ensure the data set exists and has CDN enabled. Check via the Stats API or find your data sets:

```javascript
const dataSets = await synapse.storage.findDataSets()
const dataSet = dataSets.find(ds => ds.pdpVerifierDataSetId === dataSetId)

if (!dataSet) {
  console.error('Data set not found')
}
```

### Transaction Times Out

Filecoin transactions can take 30-60 seconds. The script already waits for confirmation.

### Wrong Payer

Only the data set payer can top up. Check that your private key corresponds to the payer:

```javascript
const dataSets = await synapse.storage.findDataSets()
const dataSet = dataSets.find(ds => ds.pdpVerifierDataSetId === dataSetId)

const myAddress = await synapse.getSigner().getAddress()
console.log('Data set payer:', dataSet?.client)
console.log('Your address:', myAddress)

if (dataSet?.client?.toLowerCase() !== myAddress.toLowerCase()) {
  console.error('You are not the payer for this data set')
}
```

## Next Steps

- [Monitor Usage](monitor-usage.md) - Track your quota consumption
- [Pricing Reference](../pricing.md) - Detailed pricing information
- [Synapse SDK Documentation](https://docs.filecoin.cloud/getting-started/) - Get started with the Synapse SDK
