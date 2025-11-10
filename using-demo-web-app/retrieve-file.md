# Retrieve File

Now that you have uploaded a file to the Filecoin Onchain Cloud, with the Filecoin Beam flag set to true,  you can retrieve your file using Filecoin Beam. To do so, you will need two pieces of information:

* Your wallet address (a hex-encoded string prefixed with `0x`).
* The CID of the file (the CommP value starting with `baga`) which you kept hold of after the previous upload step.

Fill those values in the URL template below to get your unique URL for downloading the file.

Copy

```
https://{your-wallet-address}.calibration.filbeam.io/{CID}
```

**Example URL:**

{% embed url="https://0xd76b6e5e016d77123028c7ae439f631cf65535db.calibration.filbeam.io/bafkzcibeqtqqyeavuhiupl5dxo7baqx6iimwvcuomz7tmtuakjelsfsldnjn5dwggy" %}

Alternatively, you can fetch using the Synapse SDK, as shown in the previous section.
