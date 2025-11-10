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

# Set Up Your Wallet

In what follows, we may refer to the services that you can use in the Filecoin Onchain Cloud as the "Filecoin Services".

In order to use Filecoin Beam, you have to first store data on the Filecoin Onchain Cloud. The Filecoin Service that allows you to do this is called the Filecoin Warm Storage Service (FWSS). The onchain proof that guarantees this storage is called "Proof of Data Possession" (PDP). You may see all of these terms used in the following docs and in other docs for the Filecoin Onchain Cloud.

Before you can interact with FWSS to store your data, you need to create a wallet, obtain a small amount of Filecoin to pay for gas fees and enough USDFC tokens to pay for the FWSS storage & Filecoin Beam data delivery. You can do this on both Calibration Net and Mainnet.

### Calibration Net

1. Configure your wallet (e.g. Metamask) for Filecoin Calibration testnet ([Calibration docs](https://docs.filecoin.io/networks/calibration), [Metamask setup instructions](https://docs.filecoin.io/basics/assets/metamask-setup)).
2. Get tFIL. You can use one of the following faucets:
   * [Calibration Faucet - Chainsafe](https://faucet.calibnet.chainsafe-fil.io/)
   * [Calibration Faucet - Zondax](https://beryx.zondax.ch/faucet/)
   * [Calibration Faucet - Forest Explorer](https://forest-explorer.chainsafe.dev/faucet/calibnet)
3. Get testnet USDFC tokens using one of the following ways:
   * [Mint new USDFC tokens](https://docs.secured.finance/usdfc-stablecoin/getting-started/getting-test-usdfc-on-testnet)
   * Get USDFC tokens from the [Chainsafe Faucet](https://forest-explorer.chainsafe.dev/faucet/calibnet_usdfc)

### Mainnet

1. Configure your wallet (e.g. Metamask) for Filecoin Mainnet ([Mainnet docs](https://docs.filecoin.io/networks/mainnet), [Metamask setup instructions](https://docs.filecoin.io/basics/assets/metamask-setup)).
2. Get some FIL. This can be purchased from a CEX or otherwise.
3. Get Mainnet USDFC tokens using [Squidrouter](https://squidrouter.com/) or other supported DEXs
