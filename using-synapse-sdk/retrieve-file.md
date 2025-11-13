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

# Retrieve File

Now that the file is uploaded, we are going to see how we can download it (retrieve it) through Filecoin Beam.

We recommend using `synapse.storage.download(pieceCid)` API to retrieve content from Filecoin Beam. This API verifies that the retrieved content matches the requested CID.

```javascript
const downloadedData = await synapse.storage.download(cid)
```

Alternatively, if you want to perform content verification yourself (or skip it), you can retrieve the content using [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch). Below we show the calibration&#x20;

```javascript
const clientAddress = await synapse.getSigner().getAddress()
const url = `https://${clientAddress}.calibration.filbeam.io/${cid}`
// mainnet URL: const url = `https://${clientAddress}.filbeam.io/${cid}`
const res = await fetch(url)
if (!res.ok) {
  throw new Error(`Cannot retrieve ${cid}: ${res.status}`)
}
const downloadedData = await res.arrayBuffer()
// TODO: verify that `downloadedData` hashes to the digest in `cid`
```

