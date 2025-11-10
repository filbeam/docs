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

# Upload File

```javascript
// Read the file into an in-memory buffer
const fileData = await readFile(filePath)

// Run preflight checks
const preflight = await storageContext.preflightUpload(fileData.length)

if (!preflight.allowanceCheck.sufficient) {
  // The Filecoin Services deal is not sufficient
  // You need to increase the allowance, e.g. via the web app
  throw new Error(`Insufficient allowances: ${preflight.allowanceCheck.message}`)
}

const uploadResult = await storageContext.upload(fileData, {
  onPieceConfirmed: (pieceIds) => {
    // New callback - only called with updated servers
    console.log('✓ Piece addition confirmed on-chain!')
    console.log(`  Assigned piece IDs: ${pieceIds.join(', ')}`)
  },
  // Learn about additional callbas in the docs
})

const cid = uploadResult.pieceCid
// You will need `cid` to retrieve the file later
```
