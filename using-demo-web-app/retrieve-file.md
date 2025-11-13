# Retrieve File

Now that you have uploaded a file to the Filecoin Onchain Cloud, with the Filecoin Beam flag set to true,  you can retrieve your file using Filecoin Beam.&#x20;

In the app UI, you can click the downlaod icon next to the file and it will download it over Filecoin Beam.



To undertand the URL, it includes two pieces of information:

* Your wallet address (a hex-encoded string prefixed with `0x`).
* The CID of the file (the CommP value starting with `baga`) which you kept hold of after the previous upload step.

Fill those values in the URL template below to get your unique URL for downloading the file.

#### Mainnet

```
https://{your-wallet-address}.filbeam.io/{CID}
```

#### Calibration Net

```
https://{your-wallet-address}.calibration.filbeam.io/{CID}
```

Alternatively, you can fetch using the Synapse SDK, as shown in the previous section.
