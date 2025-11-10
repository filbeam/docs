# Upload File

Once you have completed the initial setup, you can switch to the Upload File tab and drop some files into your browser to upload them to Filecoin!

<figure><img src="https://images.spr.so/cdn-cgi/imagedelivery/j42No7y-dcokJuNgXeA0ig/86575f8a-2239-4107-a601-2e4f08228c51/Screenshot_2025-06-18_at_15.17.07/w=3840,quality=90,fit=scale-down" alt=""><figcaption></figcaption></figure>

1. If this is your first file upload, you will need to approve a CreateProofSet signature request. Filecoin PDP ProofSets are similar to Buckets you may know from Web2 storage offerings. In your wallet, check the transaction details - the `withCDN` parameter must be set to `true`.
2. After a file is uploaded to the storage provider, it must be linked to your ProofSet. To establish the link, you need to approve an AddRoots signature request in your wallet.
3. After the blockchain confirms the transaction, your file is stored on Filecoin and ready for retrieval. Take a note of the CommP value shown in File Upload Details - that’s the CID you will use for downloads.
