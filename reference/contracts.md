# Smart Contracts Reference

This reference documents the smart contracts used by FilBeam for payment settlement.

## Contract Addresses

### Calibration Testnet

| Contract | Address |
|----------|---------|
| FilecoinBeamOperator | Check deployment config |
| Filecoin Pay | Check [FOC docs](https://docs.filecoin.cloud) |
| Warm Storage Service | Check [FOC docs](https://docs.filecoin.cloud) |

**Note:** Contract addresses may change during testnet development. Always verify addresses from official sources.

## FilecoinBeamOperator Contract

The FilecoinBeamOperator contract manages payment rail settlement for FilBeam CDN services.

> **Note:** For complete documentation including all methods and code examples, see the [FilBeamOperator Contract Reference](filbeam-operator.md).

### ABI

```json
[
  {
    "type": "function",
    "name": "settleCDNPaymentRails",
    "inputs": [
      {
        "name": "dataSetIds",
        "type": "uint256[]",
        "internalType": "uint256[]"
      }
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  },
  {
    "type": "function",
    "name": "settleCacheMissPaymentRails",
    "inputs": [
      {
        "name": "dataSetIds",
        "type": "uint256[]",
        "internalType": "uint256[]"
      }
    ],
    "outputs": [],
    "stateMutability": "nonpayable"
  }
]
```

### Methods

#### settleCDNPaymentRails

Settles the CDN rail for the specified data sets, transferring funds to FilBeam.

```solidity
function settleCDNPaymentRails(uint256[] calldata dataSetIds) external
```

**Parameters:**
- `dataSetIds`: Array of data set IDs to settle

**Access:** Public (anyone can call)

**Usage:** This method is called automatically by the FilBeam payment-settler worker. Anyone can call this to trigger settlement, but the funds flow to the designated payee (FilBeam operator).

---

#### settleCacheMissPaymentRails

Settles cache-miss payment rails for the specified data sets. Storage providers call this to claim their earnings.

```solidity
function settleCacheMissPaymentRails(uint256[] calldata dataSetIds) external
```

**Parameters:**
- `dataSetIds`: Array of data set IDs to settle

**Access:** Public (anyone can call, but funds flow to the storage provider)

**Usage Example:**

```javascript
import { createWalletClient, createPublicClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { filecoinCalibration } from 'viem/chains'

const FilecoinBeamOperatorABI = [
  {
    type: 'function',
    name: 'settleCacheMissPaymentRails',
    inputs: [{ name: 'dataSetIds', type: 'uint256[]' }],
    outputs: [],
    stateMutability: 'nonpayable'
  }
]

const FILECOIN_BEAM_OPERATOR_ADDRESS = '0x...'  // Get from config

async function settleCacheMissPayments(dataSetIds) {
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

  // Simulate first
  const { request } = await publicClient.simulateContract({
    account,
    abi: FilecoinBeamOperatorABI,
    address: FILECOIN_BEAM_OPERATOR_ADDRESS,
    functionName: 'settleCacheMissPaymentRails',
    args: [dataSetIds.map(id => BigInt(id))]
  })

  // Execute
  const hash = await walletClient.writeContract(request)

  // Wait for confirmation
  const receipt = await publicClient.waitForTransactionReceipt({ hash })

  return receipt
}

// Usage
await settleCacheMissPayments(['12345', '67890'])
```

---

## Filecoin Pay Contract

The Filecoin Pay contract manages deposits, withdrawals, and payment rails.

### Key Methods (via Synapse SDK)

These methods are wrapped by the Synapse SDK. Direct contract interaction is not typically needed.

| Method | Description |
|--------|-------------|
| `deposit` | Deposit tokens to account |
| `withdraw` | Withdraw available funds |
| `approveOperator` | Approve service to spend |
| `revokeOperator` | Revoke operator approval |

See [Synapse SDK Reference](synapse-sdk.md) for SDK method documentation.

---

## Warm Storage Service Contract

The Warm Storage Service contract manages data sets and storage operations.

### Key Methods (via Synapse SDK)

| Method | Description |
|--------|-------------|
| `createDataSet` | Create new data set |
| `addPiece` | Add piece to data set |
| `topUpCDNPaymentRails` | Add CDN egress quota |
| `terminateDataSet` | Terminate service |

See [Synapse SDK Reference](synapse-sdk.md) for SDK method documentation.

---

## Using Contracts Directly

While the Synapse SDK is recommended, you can interact with contracts directly using viem or ethers.

### viem Example

```javascript
import { createWalletClient, createPublicClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { filecoinCalibration } from 'viem/chains'

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

// Read contract
const result = await publicClient.readContract({
  address: CONTRACT_ADDRESS,
  abi: CONTRACT_ABI,
  functionName: 'someReadMethod',
  args: [arg1, arg2]
})

// Write contract
const { request } = await publicClient.simulateContract({
  account,
  address: CONTRACT_ADDRESS,
  abi: CONTRACT_ABI,
  functionName: 'someWriteMethod',
  args: [arg1, arg2]
})

const hash = await walletClient.writeContract(request)
const receipt = await publicClient.waitForTransactionReceipt({ hash })
```

### ethers.js Example

```javascript
import { ethers } from 'ethers'

const provider = new ethers.JsonRpcProvider(
  'https://api.calibration.node.glif.io/rpc/v1'
)

const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider)

const contract = new ethers.Contract(
  CONTRACT_ADDRESS,
  CONTRACT_ABI,
  wallet
)

// Read
const result = await contract.someReadMethod(arg1, arg2)

// Write
const tx = await contract.someWriteMethod(arg1, arg2)
const receipt = await tx.wait()
```
