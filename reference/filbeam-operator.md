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

# FilBeamOperator Contract Reference

Complete reference documentation for the FilBeamOperator smart contract.

## Overview

The FilBeamOperator contract manages:
- Usage recording on-chain
- Payment rail settlement (CDN and cache-miss)
- Service termination

## Contract Addresses

| Network | Address |
|---------|---------|
| Calibration | `0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF` |
| Mainnet | `0x9E90749D298C4ca43Bb468CA859Dfe167F9CdCf2` |

## Methods

### recordUsageRollups

Records CDN and cache-miss usage on-chain.

```solidity
function recordUsageRollups(
    uint256 toEpoch,
    uint256[] calldata dataSetIds,
    uint256[] calldata cdnBytesUsed,
    uint256[] calldata cacheMissBytesUsed
) external
```

**Access:** FilBeam operator controller only

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `toEpoch` | uint256 | Filecoin epoch up to which usage is reported |
| `dataSetIds` | uint256[] | Array of dataset IDs |
| `cdnBytesUsed` | uint256[] | Total CDN egress bytes per dataset |
| `cacheMissBytesUsed` | uint256[] | Cache-miss bytes per dataset |

**Notes:**
- Called by FilBeam's usage-reporter worker
- Each array must have the same length
- Epochs are 30-second intervals on Filecoin

---

### settleCDNPaymentRails

Settles the CDN rail for specified datasets, transferring funds to FilBeam.

```solidity
function settleCDNPaymentRails(uint256[] calldata dataSetIds) external
```

**Access:** Public (anyone can call)

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `dataSetIds` | uint256[] | Array of dataset IDs to settle |

**Notes:**
- Funds flow to the FilBeam operator
- Called automatically by FilBeam's payment-settler worker
- Anyone can trigger settlement, but only the designated payee receives funds

**Example:**

```javascript
import { createWalletClient, createPublicClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { filecoinCalibration } from 'viem/chains'

const FilBeamOperatorABI = [
  {
    type: 'function',
    name: 'settleCDNPaymentRails',
    inputs: [{ name: 'dataSetIds', type: 'uint256[]' }],
    outputs: [],
    stateMutability: 'nonpayable'
  }
]

const FILBEAM_OPERATOR_ADDRESS = '0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF'

const account = privateKeyToAccount(process.env.PRIVATE_KEY)

const publicClient = createPublicClient({
  chain: filecoinCalibration,
  transport: http()
})

const walletClient = createWalletClient({
  account,
  chain: filecoinCalibration,
  transport: http()
})

async function settleCDNPayments(dataSetIds) {
  const { request } = await publicClient.simulateContract({
    account,
    abi: FilBeamOperatorABI,
    address: FILBEAM_OPERATOR_ADDRESS,
    functionName: 'settleCDNPaymentRails',
    args: [dataSetIds.map(id => BigInt(id))]
  })

  const hash = await walletClient.writeContract(request)
  const receipt = await publicClient.waitForTransactionReceipt({ hash })

  return receipt
}

// Usage
await settleCDNPayments(['12345', '67890'])
```

---

### settleCacheMissPaymentRails

Settles cache-miss payment rails for specified datasets. Storage providers use this to claim their earnings.

```solidity
function settleCacheMissPaymentRails(uint256[] calldata dataSetIds) external
```

**Access:** Public (anyone can call)

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `dataSetIds` | uint256[] | Array of dataset IDs to settle |

**Notes:**
- Funds flow to the registered storage provider for each dataset
- Anyone can trigger settlement, but only the designated storage provider receives funds
- See [Settle Payment Rails](../how-to/settle-payment-rails.md) for a complete guide

**Example:**

```javascript
import { createWalletClient, createPublicClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { filecoinCalibration } from 'viem/chains'

const FilBeamOperatorABI = [
  {
    type: 'function',
    name: 'settleCacheMissPaymentRails',
    inputs: [{ name: 'dataSetIds', type: 'uint256[]' }],
    outputs: [],
    stateMutability: 'nonpayable'
  }
]

const FILBEAM_OPERATOR_ADDRESS = '0x5991E4F9fcEF4AE23959eE03638B4688A7e1EcfF'

const account = privateKeyToAccount(process.env.SP_PRIVATE_KEY)

const publicClient = createPublicClient({
  chain: filecoinCalibration,
  transport: http()
})

const walletClient = createWalletClient({
  account,
  chain: filecoinCalibration,
  transport: http()
})

async function settleCacheMissPayments(dataSetIds) {
  const { request } = await publicClient.simulateContract({
    account,
    abi: FilBeamOperatorABI,
    address: FILBEAM_OPERATOR_ADDRESS,
    functionName: 'settleCacheMissPaymentRails',
    args: [dataSetIds.map(id => BigInt(id))]
  })

  const hash = await walletClient.writeContract(request)
  const receipt = await publicClient.waitForTransactionReceipt({ hash })

  return receipt
}

// Usage
await settleCacheMissPayments(['12345', '67890'])
```

---

### terminateCDNPaymentRails

Terminates CDN service for a dataset.

```solidity
function terminateCDNPaymentRails(uint256 dataSetId) external
```

**Access:** FilBeam operator controller only

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `dataSetId` | uint256 | Dataset ID to terminate |

**Notes:**
- Called by FilBeam's terminator worker when service ends
- Handles cleanup of payment rails and final settlement

---

### setFilBeamOperatorController

Sets the FilBeam operator controller address.

```solidity
function setFilBeamOperatorController(address _filBeamOperatorController) external
```

**Access:** Owner only

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `_filBeamOperatorController` | address | New controller address |

---

### transferFwssFilBeamController

Transfers the FWSS FilBeam controller to a new address.

```solidity
function transferFwssFilBeamController(address newController) external
```

**Access:** Owner only

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `newController` | address | New FWSS controller address |

---

## Complete ABI

```json
[
  {
    "type": "function",
    "name": "recordUsageRollups",
    "inputs": [
      { "name": "toEpoch", "type": "uint256" },
      { "name": "dataSetIds", "type": "uint256[]" },
      { "name": "cdnBytesUsed", "type": "uint256[]" },
      { "name": "cacheMissBytesUsed", "type": "uint256[]" }
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "settleCDNPaymentRails",
    "inputs": [
      { "name": "dataSetIds", "type": "uint256[]" }
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "settleCacheMissPaymentRails",
    "inputs": [
      { "name": "dataSetIds", "type": "uint256[]" }
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "terminateCDNPaymentRails",
    "inputs": [
      { "name": "dataSetId", "type": "uint256" }
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "setFilBeamOperatorController",
    "inputs": [
      { "name": "_filBeamOperatorController", "type": "address" }
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "transferFwssFilBeamController",
    "inputs": [
      { "name": "newController", "type": "address" }
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  }
]
```

## Access Control Summary

| Method | Access |
|--------|--------|
| `recordUsageRollups` | FilBeam operator controller |
| `settleCDNPaymentRails` | Public |
| `settleCacheMissPaymentRails` | Public |
| `terminateCDNPaymentRails` | FilBeam operator controller |
| `setFilBeamOperatorController` | Owner |
| `transferFwssFilBeamController` | Owner |

## Related Resources

- [Usage Reporting Explained](../explanation/usage-reporting.md) - How usage data flows to this contract
- [Observe Reported Usage](../how-to/observe-reported-usage.md) - Monitor on-chain usage data
- [Settle Payment Rails](../how-to/settle-payment-rails.md) - Claim storage provider earnings
