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

# Initialize Synapse

```javascript
import { ethers } from 'ethers'
import { RPC_URLS, Synapse } from '@filoz/synapse-sdk'

// Configuration from environment
const PRIVATE_KEY = process.env.PRIVATE_KEY
const RPC_URL = process.env.RPC_URL || RPC_URLS.calibration.http

// Initialise the SDK
const synapse = await Synapse.create({
  privateKey: PRIVATE_KEY,
  rpcURL: RPC_URL,
})

// Create storage context 
const storageContext = await synapse.storage.createContext({
  withCDN: true,
  callbacks: {
    onProviderSelected: (provider) => {
      console.log(`✓ Selected service provider: ${provider.serviceProvider}`)
    },
    // Learn about additional callbas in the docs
  },
})
```
