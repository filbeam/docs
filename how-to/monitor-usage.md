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

# Monitor Usage (Clients)

This guide explains how to track your FilBeam egress usage and quota consumption using the Synapse SDK, Stats API, or the FilBeam Dashboard.

## Using Synapse SDK

### Check Data Set Quota

```javascript
import { Synapse } from '@filoz/synapse-sdk'

const synapse = await Synapse.create({
  privateKey: process.env.PRIVATE_KEY,
  network: 'calibration',
  withCDN: true,
})

const dataSetId = 0 // replace with your data set ID
const stats = await synapse.filbeamService.getDataSetStats(dataSetId)

console.log('CDN Egress Quota:', stats.cdnEgressQuota, 'bytes')
console.log('Cache Miss Quota:', stats.cacheMissEgressQuota, 'bytes')
```

### List All Data Sets

```javascript
const dataSets = await synapse.storage.getDataSets()

for (const ds of dataSets) {
  if (ds.withCDN) {
    const stats = await synapse.filbeamService.getDataSetStats(ds.id)
    console.log(`Data Set ${ds.id}:`)
    console.log(`  CDN Quota: ${formatBytes(stats.cdnEgressQuota)}`)
    console.log(`  Pieces: ${ds.pieceCount}`)
  }
}
```

## Using Stats API

The Stats API provides REST endpoints for quota and usage data.

### Get Data Set Stats

```
GET https://stats.filbeam.io/data-set/{dataSetId}
```

Response:
```json
{
  "cdnEgressQuota": "1099511627776",
  "cacheMissEgressQuota": "549755813888"
}
```

#### JavaScript Example

```javascript
async function getDataSetStats(dataSetId) {
  const response = await fetch(
    `https://stats.filbeam.io/data-set/${dataSetId}`
  )

  if (!response.ok) {
    if (response.status === 404) {
      throw new Error('Data set not found')
    }
    throw new Error(`API error: ${response.status}`)
  }

  return response.json()
}

// Usage
const stats = await getDataSetStats('12345')
console.log('CDN Quota:', stats.cdnEgressQuota)
```

### Get Payer Stats (Aggregate)

Get combined statistics across all your data sets:

```
GET https://stats.filbeam.io/payer/{payerAddress}
```

Response:
```json
{
  "totalRequests": "15234",
  "cacheMissRequests": "3847",
  "totalEgressBytes": "52428800000",
  "cacheMissEgressBytes": "13107200000",
  "remainingCDNEgressBytes": "1047483648000",
  "remainingCacheMissEgressBytes": "523741824000"
}
```

#### JavaScript Example

```javascript
async function getPayerStats(payerAddress) {
  const response = await fetch(
    `https://stats.filbeam.io/payer/${payerAddress.toLowerCase()}`
  )

  if (!response.ok) {
    throw new Error(`API error: ${response.status}`)
  }

  return response.json()
}

// Usage
const stats = await getPayerStats('0x1234...')

console.log('Total Requests:', stats.totalRequests)
console.log('Cache Miss Requests:', stats.cacheMissRequests)
console.log('Total Egress:', formatBytes(stats.totalEgressBytes))
console.log('Remaining CDN Quota:', formatBytes(stats.remainingCDNEgressBytes))
```

## Using FilBeam Dashboard

The FilBeam Dashboard provides a visual interface for monitoring your usage without writing code.

Visit your client dashboard at:

```
https://calibration.dashboard.filbeam.com/client/<your-wallet-address>
```

The dashboard displays:

- Stats for all your data sets
- Remaining CDN egress quota per data set
- Remaining cache miss egress quota per data set

{% hint style="warning" %}
**Note:** The dashboard is deployed hourly, so stats are not real-time. For real-time data, use the Stats API or Synapse SDK.
{% endhint %}

## Next Steps

- [Top Up CDN Quota](top-up-cdn-quota.md) - Add capacity when running low
- [Stats API Reference](../reference/stats-api.md) - Full API documentation
- [Quota System Explained](../explanation/quota-system.md) - Understanding dual quotas
- [Synapse SDK Documentation](https://docs.filecoin.cloud/getting-started/) - Get started with the Synapse SDK
