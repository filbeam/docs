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

We recommend using `synapse.storage.download(pieceCid)` API to retrieve content from Filecoin Beam. This API verifies that the retrieved content matches the requested CID.

Copy

```javascript
const downloadedData = await synapse.storage.download(cid)
```

Alternatively, if you want to perform content verification yourself (or skip it), you can retrieve the content using [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch).

Copy

```javascript
const clientAddress = await synapse.getSigner().getAddress()
const url = `https://${clientAddress}.calibration.filcdn.io/${cid}`
const res = await fetch(url)
if (!res.ok) {
  throw new Error(`Cannot retrieve ${cid}: ${res.status}`)
}
const downloadedData = await res.arrayBuffer()
// TODO: verify that `downloadedData` hashes to the digest in `cid`
```

**Example URL:**

[https://0xd76b6e5e016d77123028c7ae439f631cf65535db.calibration.filbeam.io/bafkzcibeqtqqyeavuhiupl5dxo7baqx6iimwvcuomz7tmtuakjelsfsldnjn5dwggy](https://0xd76b6e5e016d77123028c7ae439f631cf65535db.calibration.filbeam.io/bafkzcibeqtqqyeavuhiupl5dxo7baqx6iimwvcuomz7tmtuakjelsfsldnjn5dwggy)
