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

# Settle Payment Rails (For Storage Providers)

This guide explains how storage providers can settle their payment rails to collect earnings from cache-miss retrievals.

## Overview

When users retrieve content through FilBeam that results in a cache miss, the storage provider serves the data and earns fees. These fees accumulate in payment rails and must be settled to claim the earnings.

### How It Works

1. **User requests content** via FilBeam CDN
2. **Cache miss occurs** - content is fetched from your storage node
3. **Fees accumulate** in the cache-miss payment rail
4. **You settle** the payment rail to claim your earnings
5. **Earnings transferred** to your Filecoin Pay accounts

## Prerequisites

- Storage provider wallet with signing capability
- Node.js with viem library installed
- Data set IDs for which you're the storage provider
- tFIL for gas fees on Calibration testnet

## Contract Information

### Calibration Testnet

| Property | Value |
|----------|-------|
| Contract | FilBeamOperator |
| Address | `0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF` |
| Method | `settleCacheMissPaymentRails(uint256[] dataSetIds)` |

### Mainnet

| Property | Value |
|----------|-------|
| Contract | FilBeamOperator |
| Address | `0x9E90749D298C4ca43Bb468CA859Dfe167F9CdCf2` |
| Method | `settleCacheMissPaymentRails(uint256[] dataSetIds)` |

## Option 1: Using Cast CLI (Foundry)

The simplest way to settle payment rails is using the `cast` command from [Foundry](https://book.getfoundry.sh/).

### Install Foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

### Settle a Single Data Set

```bash
# Set your private key (or use --interactive for prompt)
export PRIVATE_KEY=0x...

# Calibration testnet
cast send 0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF \
  "settleCacheMissPaymentRails(uint256[])" "[12345]" \
  --private-key $PRIVATE_KEY \
  --rpc-url https://api.calibration.node.glif.io/rpc/v1

# Mainnet
cast send 0x9E90749D298C4ca43Bb468CA859Dfe167F9CdCf2 \
  "settleCacheMissPaymentRails(uint256[])" "[12345]" \
  --private-key $PRIVATE_KEY \
  --rpc-url https://api.node.glif.io/rpc/v1
```

### Settle Multiple Data Sets

```bash
# Batch multiple data sets in one transaction
cast send 0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF \
  "settleCacheMissPaymentRails(uint256[])" "[12345,67890,11111]" \
  --private-key $PRIVATE_KEY \
  --rpc-url https://api.calibration.node.glif.io/rpc/v1
```

### Simulate Before Sending

Use `cast call` to simulate without spending gas:

```bash
cast call 0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF \
  "settleCacheMissPaymentRails(uint256[])" "[12345]" \
  --rpc-url https://api.calibration.node.glif.io/rpc/v1
```

### Using a Ledger Hardware Wallet

```bash
cast send 0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF \
  "settleCacheMissPaymentRails(uint256[])" "[12345]" \
  --ledger \
  --rpc-url https://api.calibration.node.glif.io/rpc/v1
```

## Option 2: Using JavaScript (viem)

### Step 1: Set Up Your Environment

Install dependencies:

```bash
npm install viem
```

### Step 2: Configure the Client

```javascript
import { createWalletClient, createPublicClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { filecoinCalibration } from 'viem/chains'

// Your storage provider private key
const account = privateKeyToAccount(process.env.SP_PRIVATE_KEY)

// Create clients
const publicClient = createPublicClient({
  chain: filecoinCalibration,
  transport: http()
})

const walletClient = createWalletClient({
  account,
  chain: filecoinCalibration,
  transport: http()
})

console.log('Storage Provider Address:', account.address)
```

### Step 3: Define the Contract ABI

```javascript
const FilBeamOperatorABI = [
  {
    type: 'function',
    name: 'settleCacheMissPaymentRails',
    inputs: [
      {
        name: 'dataSetIds',
        type: 'uint256[]',
        internalType: 'uint256[]'
      }
    ],
    outputs: [],
    stateMutability: 'nonpayable'
  }
]

// Contract addresses
const FILBEAM_OPERATOR_CALIBRATION = '0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF'
const FILBEAM_OPERATOR_MAINNET = '0x9E90749D298C4ca43Bb468CA859Dfe167F9CdCf2'
```

### Step 4: Settle Payment Rails

```javascript
async function settleCacheMissPaymentRails(dataSetIds) {
  console.log(`Settling payment rails for ${dataSetIds.length} data sets`)

  // Simulate the transaction first
  const { request } = await publicClient.simulateContract({
    account,
    abi: FilBeamOperatorABI,
    address: FILBEAM_OPERATOR_CALIBRATION,
    functionName: 'settleCacheMissPaymentRails',
    args: [dataSetIds.map(id => BigInt(id))]
  })

  // Execute the transaction
  const hash = await walletClient.writeContract(request)
  console.log('Transaction submitted:', hash)

  // Wait for confirmation
  const receipt = await publicClient.waitForTransactionReceipt({ hash })
  console.log('Transaction confirmed in block:', receipt.blockNumber)

  return receipt
}

// Usage
const dataSetIds = ['12345', '67890', '11111']
await settleCacheMissPaymentRails(dataSetIds)
```

## Complete Settlement Script

```javascript
import { createWalletClient, createPublicClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { filecoinCalibration } from 'viem/chains'

// Configuration
const SP_PRIVATE_KEY = process.env.SP_PRIVATE_KEY
const FILBEAM_OPERATOR_ADDRESS = '0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF'  // Calibration

const FilBeamOperatorABI = [
  {
    type: 'function',
    name: 'settleCacheMissPaymentRails',
    inputs: [{ name: 'dataSetIds', type: 'uint256[]' }],
    outputs: [],
    stateMutability: 'nonpayable'
  }
]

async function main() {
  // Setup
  const account = privateKeyToAccount(SP_PRIVATE_KEY)

  const publicClient = createPublicClient({
    chain: filecoinCalibration,
    transport: http()
  })

  const walletClient = createWalletClient({
    account,
    chain: filecoinCalibration,
    transport: http()
  })

  console.log('=== FilBeam Payment Rail Settlement ===')
  console.log('Storage Provider:', account.address)

  // Data sets to settle (replace with your actual data set IDs)
  const dataSetIds = [
    '12345',
    '67890'
  ]

  if (dataSetIds.length === 0) {
    console.log('No data sets to settle')
    return
  }

  console.log(`\nSettling ${dataSetIds.length} data sets:`, dataSetIds)

  try {
    // Simulate first to catch errors
    console.log('\nSimulating transaction...')
    const { request } = await publicClient.simulateContract({
      account,
      abi: FilBeamOperatorABI,
      address: FILBEAM_OPERATOR_ADDRESS,
      functionName: 'settleCacheMissPaymentRails',
      args: [dataSetIds.map(id => BigInt(id))]
    })

    console.log('Simulation successful, submitting transaction...')

    // Submit transaction
    const hash = await walletClient.writeContract(request)
    console.log('Transaction hash:', hash)

    // Wait for confirmation
    console.log('Waiting for confirmation...')
    const receipt = await publicClient.waitForTransactionReceipt({
      hash,
      timeout: 120_000  // 2 minute timeout
    })

    if (receipt.status === 'success') {
      console.log('\nSettlement successful!')
      console.log('Block number:', receipt.blockNumber)
      console.log('Gas used:', receipt.gasUsed)
    } else {
      console.log('\nTransaction failed')
    }
  } catch (error) {
    console.error('\nSettlement failed:', error.message)

    if (error.message.includes('insufficient funds')) {
      console.log('Hint: You need more tFIL for gas fees')
    }
  }
}

main().catch(console.error)
```

## Best Practices

### When to Settle

We recommend settling payment rails at least once per week. More frequent settlements incur unnecessary gas fees, while less frequent settlements create a risk that payers can claim their lockup after terminating a payment rail.

### Gas Optimization

Batch multiple data sets in a single transaction:

```javascript
// Good: Batch settlement (one transaction)
await settleCacheMissPaymentRails(['12345', '67890', '11111'])

// Avoid: Individual settlements (multiple transactions = more gas)
await settleCacheMissPaymentRails(['12345'])
await settleCacheMissPaymentRails(['67890'])
await settleCacheMissPaymentRails(['11111'])
```

### Monitoring Settlements

Track your settlements:

```javascript
async function logSettlement(dataSetIds, receipt) {
  const log = {
    timestamp: new Date().toISOString(),
    dataSetIds,
    transactionHash: receipt.transactionHash,
    blockNumber: receipt.blockNumber,
    gasUsed: receipt.gasUsed.toString(),
    status: receipt.status
  }

  // Save to file or database
  console.log('Settlement log:', JSON.stringify(log, null, 2))
}
```

## Troubleshooting

### "Transaction reverted" Error

- Check that the data sets have active payment rails with accumulated funds
- Verify the contract address is correct
- Ensure the data sets exist and have CDN enabled

**Note:** The settlement function is public - anyone can call it. However, the funds always flow to the designated storage provider, so there's no benefit to others calling it.

### "Insufficient funds" Error

You need tFIL for gas: Get testnet FIL from faucet:

https://faucet.calibnet.chainsafe-fil.io/

### "Data set not found" Error

- Verify the data set ID is correct
- Ensure the data set has CDN enabled
- Check that you're the registered storage provider

## Related Resources

- [Payment Model](../explanation/payment-model.md) - Understanding pay-per-byte billing
- [FilBeamOperator](../reference/filbeam-operator.md) - FilBeamOperator contract addresses and ABIs
