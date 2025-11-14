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

After the initial payment setup, we must initialize the Synapse SDK.

The important bit to look at in the following is the `withCDN` flag in the storage context. Setting this to true means you are making your deal with Filecoin Beam for reliable and incentivized data retrieval.

Conversely, if this flag is set to `false`, then you must fetch your data directly from the Storage Provider and there are no incentives included for the SP. This means potentially unreliable retrievability and long distance round trips to fetch data.

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
    // Learn about additional callbacks in the docs
  },
})
```
