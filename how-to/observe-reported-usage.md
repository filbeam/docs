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

# Observe Reported Usage (Storage Providers)

This guide explains how storage providers can observe and verify the usage data that FilBeam reports on-chain.

## Overview

FilBeam periodically reports CDN usage to the blockchain via the `FilBeamOperator.recordUsageRollups` function. Storage providers can monitor these transactions to verify that reported cache-miss bytes match their delivery logs.

## Important Note

**Only traffic proxied through FilBeam is reported on-chain.**

If users retrieve content directly from your storage node (bypassing FilBeam), that traffic:
- Is NOT reported to the blockchain
- Is NOT subject to FilBeam billing
- Will NOT appear in `recordUsageRollups` transactions

## Contract Information

### Addresses

| Network | FilBeamOperator Address |
|---------|------------------------|
| Calibration | `0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF` |
| Mainnet | `0x9E90749D298C4ca43Bb468CA859Dfe167F9CdCf2` |

### Method Signature

```solidity
function recordUsageRollups(
    uint256 toEpoch,
    uint256[] calldata dataSetIds,
    uint256[] calldata cdnBytesUsed,
    uint256[] calldata cacheMissBytesUsed
) external
```

## Method 1: Monitor Transactions

Watch for `recordUsageRollups` transactions and decode the calldata.

### Step 1: Set Up Your Environment

```bash
npm install viem
```

### Step 2: Define the ABI

```javascript
const FilBeamOperatorABI = [
  {
    type: 'function',
    name: 'recordUsageRollups',
    inputs: [
      { name: 'toEpoch', type: 'uint256' },
      { name: 'dataSetIds', type: 'uint256[]' },
      { name: 'cdnBytesUsed', type: 'uint256[]' },
      { name: 'cacheMissBytesUsed', type: 'uint256[]' }
    ],
    outputs: [],
    stateMutability: 'nonpayable'
  }
]
```

### Step 3: Decode Transaction Data

```javascript
import { createPublicClient, http, decodeFunctionData } from 'viem'
import { filecoinCalibration } from 'viem/chains'

const FILBEAM_OPERATOR_ADDRESS = '0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF'

const publicClient = createPublicClient({
  chain: filecoinCalibration,
  transport: http()
})

async function decodeUsageReport(txHash) {
  // Get transaction
  const tx = await publicClient.getTransaction({ hash: txHash })

  // Verify tx was sent to the FilBeamOperator contract address
  if (tx.to?.toLowerCase() !== FILBEAM_OPERATOR_ADDRESS.toLowerCase()) {
    throw new Error('Transaction not sent to the FilBeamOperator contract address')
  }

  // Decode the function call
  const decoded = decodeFunctionData({
    abi: FilBeamOperatorABI,
    data: tx.input
  })

  if (decoded.functionName !== 'recordUsageRollups') {
    throw new Error('Not a recordUsageRollups transaction')
  }

  const [toEpoch, dataSetIds, cdnBytesUsed, cacheMissBytesUsed] = decoded.args

  return {
    toEpoch: toEpoch.toString(),
    reports: dataSetIds.map((id, i) => ({
      dataSetId: id.toString(),
      cdnBytesUsed: cdnBytesUsed[i].toString(),
      cacheMissBytesUsed: cacheMissBytesUsed[i].toString()
    }))
  }
}

// Usage
const report = await decodeUsageReport('0x...')
console.log('Usage Report:', report)
```

### Step 4: Watch for New Transactions

```javascript
async function watchUsageReports(onReport) {
  console.log('Watching for usage reports...')

  // Get latest block
  let lastBlock = await publicClient.getBlockNumber()

  setInterval(async () => {
    const currentBlock = await publicClient.getBlockNumber()

    if (currentBlock > lastBlock) {
      // Get transactions in new blocks
      for (let blockNum = lastBlock + 1n; blockNum <= currentBlock; blockNum++) {
        const block = await publicClient.getBlock({
          blockNumber: blockNum,
          includeTransactions: true
        })

        for (const tx of block.transactions) {
          if (tx.to?.toLowerCase() === FILBEAM_OPERATOR_ADDRESS.toLowerCase()) {
            try {
              const report = await decodeUsageReport(tx.hash)
              onReport(report, tx.hash, blockNum)
            } catch (e) {
              // Not a recordUsageRollups transaction
            }
          }
        }
      }

      lastBlock = currentBlock
    }
  }, 30000) // Check every 30 seconds (1 epoch)
}

// Start watching
watchUsageReports((report, txHash, blockNum) => {
  console.log(`\n=== New Usage Report ===`)
  console.log(`Block: ${blockNum}`)
  console.log(`Transaction: ${txHash}`)
  console.log(`Reported up to epoch: ${report.toEpoch}`)
  console.log(`Datasets:`)
  for (const r of report.reports) {
    console.log(`  Dataset ${r.dataSetId}:`)
    console.log(`    CDN Bytes: ${formatBytes(r.cdnBytesUsed)}`)
    console.log(`    Cache Miss Bytes: ${formatBytes(r.cacheMissBytesUsed)}`)
  }
})
```

## Method 2: Query Historical Transactions

Fetch past usage reports for analysis.

```javascript
async function getHistoricalUsageReports(fromBlock, toBlock) {
  const reports = []

  // Note: For large ranges, paginate or use an indexer
  for (let blockNum = fromBlock; blockNum <= toBlock; blockNum++) {
    const block = await publicClient.getBlock({
      blockNumber: BigInt(blockNum),
      includeTransactions: true
    })

    for (const tx of block.transactions) {
      if (tx.to?.toLowerCase() === FILBEAM_OPERATOR_ADDRESS.toLowerCase()) {
        try {
          const report = await decodeUsageReport(tx.hash)
          reports.push({
            blockNumber: blockNum,
            txHash: tx.hash,
            ...report
          })
        } catch (e) {
          // Not a recordUsageRollups transaction
        }
      }
    }
  }

  return reports
}
```

## Method 3: Filter by Your Dataset IDs

If you only care about specific datasets:

```javascript
async function getUsageForDatasets(dataSetIds, fromBlock, toBlock) {
  const myDataSetIds = new Set(dataSetIds.map(id => id.toString()))
  const allReports = await getHistoricalUsageReports(fromBlock, toBlock)

  return allReports.map(report => ({
    ...report,
    reports: report.reports.filter(r => myDataSetIds.has(r.dataSetId))
  })).filter(report => report.reports.length > 0)
}

// Usage
const myUsage = await getUsageForDatasets(['12345', '67890'], 1000000n, 1001000n)
```

## Understanding the Data

For detailed field documentation, see [FilBeamOperator Contract Reference](../reference/filbeam-operator.md#recordusagerollups).

**Quick reference:**

| Field | Description |
|-------|-------------|
| `toEpoch` | Filecoin epoch up to which usage is reported |
| `dataSetIds` | Array of dataset IDs in this report |
| `cdnBytesUsed` | Total CDN egress per dataset (cache hits + misses) |
| `cacheMissBytesUsed` | Cache miss bytes per dataset (your compensation basis) |

The complete monitoring script below includes helper functions for epoch conversion and byte formatting.

## Complete Monitoring Script

```javascript
import { createPublicClient, http, decodeFunctionData } from 'viem'
import { filecoinCalibration } from 'viem/chains'

// Configuration
const FILBEAM_OPERATOR_ADDRESS = '0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF'
const MY_DATASET_IDS = ['12345', '67890'] // Your dataset IDs

const FilBeamOperatorABI = [
  {
    type: 'function',
    name: 'recordUsageRollups',
    inputs: [
      { name: 'toEpoch', type: 'uint256' },
      { name: 'dataSetIds', type: 'uint256[]' },
      { name: 'cdnBytesUsed', type: 'uint256[]' },
      { name: 'cacheMissBytesUsed', type: 'uint256[]' }
    ],
    outputs: [],
    stateMutability: 'nonpayable'
  }
]

// Genesis timestamps differ by network
const CALIBRATION_GENESIS_TIMESTAMP = 1667326380000 // Calibration testnet
const MAINNET_GENESIS_TIMESTAMP = 1598306400000 // Mainnet
const GENESIS_TIMESTAMP = CALIBRATION_GENESIS_TIMESTAMP // Using Calibration
const EPOCH_DURATION = 30000

const publicClient = createPublicClient({
  chain: filecoinCalibration,
  transport: http()
})

function formatBytes(bytes) {
  const units = ['B', 'KiB', 'MiB', 'GiB', 'TiB']
  let value = Number(bytes)
  let unitIndex = 0
  while (value >= 1024 && unitIndex < units.length - 1) {
    value /= 1024
    unitIndex++
  }
  return `${value.toFixed(2)} ${units[unitIndex]}`
}

function epochToDate(epoch) {
  return new Date(GENESIS_TIMESTAMP + (Number(epoch) * EPOCH_DURATION))
}

async function main() {
  const myIds = new Set(MY_DATASET_IDS)
  let lastBlock = await publicClient.getBlockNumber()

  console.log('=== FilBeam Usage Monitor ===')
  console.log(`Monitoring datasets: ${MY_DATASET_IDS.join(', ')}`)
  console.log(`FilBeamOperator: ${FILBEAM_OPERATOR_ADDRESS}`)
  console.log(`Starting from block: ${lastBlock}`)
  console.log('')

  setInterval(async () => {
    try {
      const currentBlock = await publicClient.getBlockNumber()

      for (let blockNum = lastBlock + 1n; blockNum <= currentBlock; blockNum++) {
        const block = await publicClient.getBlock({
          blockNumber: blockNum,
          includeTransactions: true
        })

        for (const tx of block.transactions) {
          if (tx.to?.toLowerCase() === FILBEAM_OPERATOR_ADDRESS.toLowerCase()) {
            try {
              const decoded = decodeFunctionData({
                abi: FilBeamOperatorABI,
                data: tx.input
              })

              if (decoded.functionName === 'recordUsageRollups') {
                const [toEpoch, dataSetIds, cdnBytesUsed, cacheMissBytesUsed] = decoded.args

                // Filter for my datasets
                const myReports = dataSetIds
                  .map((id, i) => ({
                    dataSetId: id.toString(),
                    cdnBytes: cdnBytesUsed[i],
                    cacheMissBytes: cacheMissBytesUsed[i]
                  }))
                  .filter(r => myIds.has(r.dataSetId))

                if (myReports.length > 0) {
                  console.log(`\n[${new Date().toISOString()}] New Usage Report`)
                  console.log(`  Block: ${blockNum}`)
                  console.log(`  TX: ${tx.hash}`)
                  console.log(`  Period ends: ${epochToDate(toEpoch).toISOString()}`)

                  for (const r of myReports) {
                    console.log(`  Dataset ${r.dataSetId}:`)
                    console.log(`    CDN: ${formatBytes(r.cdnBytes)}`)
                    console.log(`    Cache Miss: ${formatBytes(r.cacheMissBytes)}`)
                  }
                }
              }
            } catch (e) {
              // Not a valid recordUsageRollups call
            }
          }
        }
      }

      lastBlock = currentBlock
    } catch (error) {
      console.error('Error checking blocks:', error.message)
    }
  }, 30000)
}

main().catch(console.error)
```

## Verifying Against Your Logs

To verify reported usage matches your delivery:

1. **Track your deliveries**: Log every request you serve for FilBeam
2. **Match epochs**: Group your logs by the same epoch boundaries
3. **Compare**: Your cache-miss bytes should match `cacheMissBytesUsed`

```javascript
// Example verification
function verifyUsage(myLogs, reportedUsage) {
  const myTotal = myLogs.reduce((sum, log) => sum + log.bytes, 0n)
  const reported = BigInt(reportedUsage.cacheMissBytesUsed)

  const diff = myTotal - reported
  const diffPercent = Number(diff * 100n / myTotal)

  console.log('My tracked bytes:', formatBytes(myTotal))
  console.log('Reported bytes:', formatBytes(reported))
  console.log('Difference:', formatBytes(diff), `(${diffPercent.toFixed(2)}%)`)

  if (Math.abs(diffPercent) > 5) {
    console.warn('⚠️  Significant discrepancy detected!')
  } else {
    console.log('✅ Usage matches within tolerance')
  }
}
```

## Related Resources

- [Usage Reporting Explained](../explanation/usage-reporting.md) - How the system works
- [FilBeamOperator Reference](../reference/filbeam-operator.md) - Contract documentation
- [Settle Payment Rails](settle-payment-rails.md) - Claim your earnings
