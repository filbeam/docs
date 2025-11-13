# Upload File

Now that you hae added some funds, you click "Create Data Set" to add a file. This opens a pop up window where you choose a Storage Provider with which to stoe the data and a toggle for whether or no to include FIlecoin Beam for reliable, fast, economically aligned access. We encourage you to opt in to Filecoin Beam and refer you to the [benefits](../benefits.md) page to understand why.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>







1. Now click "Create". If this is your first file upload, you will need to approve a CreateProofSet signature request. Filecoin PDP ProofSets are similar to Buckets you may know from Web2 storage offerings. In your wallet, check the transaction details - the `withCDN` parameter must be set to `true` in order to enable Filecoin Beam data delivery. Now you must wait for the transaction to reach finality, perhaps 30 seconds.
2. With the proof set created, you are ready to upload a file. Click on the upload dialog and upload a file in the UI. Then click "Upload". This will require another signature. Again you will need to wait for chain finality here.
3. After the blockchain confirms the transaction, your file is stored on Filecoin Onchain Cloud and ready for retrieval. Take a note of the CommP value shown in File Upload Details - that’s the CID you will use for downloads.
