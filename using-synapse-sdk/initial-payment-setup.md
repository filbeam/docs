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

# Initial Payment Setup

You need to set up payments before you can upload files to Filecoin PDP.

**Using Synapse SDK API**

```javascript
import { ethers } from 'ethers'
import { RPC_URLS, Synapse } from '@filoz/synapse-sdk'

// Configuration from environment
const PRIVATE_KEY = process.env.PRIVATE_KEY
const RPC_URL = process.env.RPC_URL || RPC_URLS.calibration.http

// Initialize SDK
const synapse = await Synapse.create({
  privateKey: PRIVATE_KEY,
  rpcURL: RPC_URL,
});

// Deposit USDFC tokens (one-time setup)
const amount = ethers.parseUnits("10", 18); // 10 USDFC
await synapse.payments.deposit(amount);

// Approve the Warm Storage service for automated payments
// The SDK automatically uses the correct service address for your network
const warmStorageAddress = synapse.getWarmStorageAddress();
await synapse.payments.approveService(
  warmStorageAddress,
  ethers.parseUnits("1", 18), // Rate allowance: 1 USDFC per epoch
  ethers.parseUnits("10", 18), // Lockup allowance: 10 USDFC total
  86400n, // Max lockup period: 30 days (in epochs)
);
```
