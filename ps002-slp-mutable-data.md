# SLP Token Mutable Data
## Permissionless Software Foundation Specification 002 (PS002)

### Specification version: 1.0.0
### Date originally published: July 20, 2020
### Date last updated: July 20, 2020

## Authors
Chris Troutner

## Acknowledgements
- James Cramer created the [Bitcoin Files Specification](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md) leveraged by this document.
- This document builds on top of the [SLP Token Type 1 Protocol](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md).
- This document builds on top of the [PS001 Media Sharing Protocol](./ps001-media-sharing.md)

## 1. Introduction
The Simple Ledger Protocol (SLP) for tokens includes a `document hash` field in the [specification for the Genesis transaction](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md#genesis---token-genesis-transaction). This field is intended to hold a TXID for a Bitcoin Cash transaction. This TXID was originally intended to point to an on-chain file uploaded with the [Bitcoin Files Specification](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md).

This document specifies how to leverage the `document hash` field when creating an SLP token to point to mutable data (data that can change over time). This provides many new use cases, such security tokens, tracking state of a tokenized asset (like a video game character), and many others. This specification is a general approach for wallets to discover additional mutable data associated with the token.

## 2. Protocol Overview

There are four parts to *encoding a pointer* to mutable data and attaching it to the Genesis transaction for creating a new SLP token:

1. Create a [MSP bitcoincash address](./ps001-media-sharing.md).
2. Insert the address into JSON file
3. Upload JSON file to the chain using Bitcoin Files Protocol
4. Include TXID in document hash when creating a token.

Likewise, there are four steps to unwind the process when wallet software wants to *parse and download* the data:

1. Look up the document hash in the token's Genesis transaction.
2. Retrieve the BFP file from the blockchain.
3. Open the JSON document and retrieve the MSP bitcoin cash address.
4. Download the data pointed to by the MSP address.

## 3. Pointing to Mutable Data

### 3.1 Create An Address
The first step is to create a system for pointing to mutable data. This specification leverages the [PS001 Media Sharing Protocol](./ps001-media-sharing.md) and the [memo-push](https://github.com/christroutner/memo-push) app to point at changing data, stored on the IPFS network, as described on [UncensorablePublishing.com](https://uncensorablepublishing.com/).

### 3.2 Insert Address into JSON File
Once a Bitcoin Cash address has been created, that address can be written into a JSON document such as this:

```json
{
  "mspAddress": "bitcoincash:qpt5kra7j4dhqkws6l4g95cxxemyzueyavyntd39cz"
}
```

### 3.3 Upload JSON File On-Chain
The JSON file can then be uploaded to the Bitcoin Cash blockchain using the [Bitcoin Files Specification](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md). [Here is example code](https://github.com/Permissionless-Software-Foundation/bch-js-examples/tree/master/applications/bfp) for writing such a file using BFP.

### 3.4 Use TXID in Document Hash Field
The final step is to create a new SLP token. This could be of Token Type 1, or an NFT. Any SLP token has a `document hash` field in the Genesis transaction. The TXID output from the BFP file upload can be inserted into this field.
