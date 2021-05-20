# SLP Token Mutable Data

## Permissionless Software Foundation Specification 002 (PS002)

### Specification version: 1.0.0

### Date originally published: July 20, 2020

### Date last updated: May 20, 2021

## Authors

Chris Troutner

## Acknowledgements

- This document builds on top of the [SLP Token Type 1 Protocol](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md).
- This document builds on top of the [PS001 Media Sharing Protocol](./ps001-media-sharing.md)

## 1. Introduction

The Simple Ledger Protocol (SLP) for tokens includes a `token_document_hash` field in the [specification for the Genesis transaction](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md#genesis---token-genesis-transaction). This field is intended to hold a TXID for a Bitcoin Cash transaction. This TXID was originally intended to point to an on-chain file uploaded with the [Bitcoin Files Specification](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md), but can be used with any arbitrary BCH transaction.

This document specifies how to leverage the `token_document_hash` field when creating an SLP token, to point to mutable data (data that can change over time). This provides many new use cases. Here are a few examples:

- Token Icons
- Tracking state of tokenized assets (like a video game character)
- Security tokens compliant with arbitrary regulation rules.
- NFTs that leverage both mutable and immutable data.

This specification is a general approach for wallets to discover additional mutable data associated with the token.

## 2. Protocol Overview

There are four parts to _encoding a pointer_ to mutable data and attaching it to the Genesis transaction when creating a new SLP token:

1. Create a [MSP bitcoincash address](./ps001-media-sharing.md).
2. Insert the address into a JSON file.
3. Upload the JSON file to the blockchain, which generates a TXID.
4. Include the TXID in the `token_document_hash` field when creating a token.

Likewise, there are four steps to unwind the process when wallet software wants to _parse and download_ the data:

1. Look up the `token_document_hash` field in the token's Genesis transaction.
2. Retrieve the JSON data from the blockchain.
3. Retrieve the MSP bitcoin cash address from the JSON.
4. Download the data pointed to by the MSP address.

## 3. Pointing to Mutable Data

### 3.1 Create An Address

The first step is to create a system for pointing to mutable data. This specification leverages the [PS001 Media Sharing Protocol](./ps001-media-sharing.md) and the [memo-push](https://github.com/christroutner/memo-push) app to point at changing data, stored on the IPFS network, as described on [UncensorablePublishing.com](https://uncensorablepublishing.com/).

### 3.2 Insert Address into JSON File

Once a Bitcoin Cash address has been created, that address can be written into a JSON document. Here is an example:

```json
{
  "mspAddress": "bitcoincash:qpt5kra7j4dhqkws6l4g95cxxemyzueyavyntd39cz"
}
```

### 3.3 Upload JSON File On-Chain

The JSON data above can be uploaded to the blockchain using the OP_RETURN of a single transaction. [Here is example code](https://github.com/Permissionless-Software-Foundation/bch-js-examples/tree/master/low-level/op-return) for working with OP_RETURN. Also, the [memo-push](https://github.com/christroutner/memo-push) library can be used for this.

- [Here is an example transaction](https://explorer.bitcoin.com/bch/tx/4b7d5eb0d27157c2862e0d507f6ea9438fa94230999690233610cc20d9b584f7) encoding an address in JSON, using the [Memo.cash protocol](https://memo.cash/protocol).

### 3.4 Use TXID in Document Hash Field

The final step is to create a new SLP token. This could be of Token Type 1, or an NFT. Any SLP token has a `token_document_hash` field in the Genesis transaction. The TXID output from step 3.3 can be inserted into this field.

### 3.5 Updating Data

The data pointed to by the MSP address can be any kind of arbitrary data, however [JSON-LD](https://json-ld.org/) formatted Linked Data following the [Schema.org](https://schema.org/) schema is recommended. Here is an example of what a token icon might look like:

```
{
  "@context": "https://schema.org",
  "@type": "ImageObject",
  "author": "Chris Troutner",
  "contentUrl": "https://hub.textile.io/ipfs/bafkreiasjlveyusmkgludz6vopzg5t5g3lefgtu5oudoawjrcttmgwjea4",
  "datePublished": "2021-05-20",
  "description": "PSF Logo",
  "name": "psf-logo.png"
}
```

This state can be updated by creating a new JSON document, uploading it to IPFS, and writing a new transaction with the new IPFS CID to the blockchain via the [MSP protocol](./ps001-media-sharing.md).

## 4. Reading Mutible Data

Wallet software can unwind the process to read the data or 'state' of the token by following these steps.

### 4.1 Retrieve the MSP Address

When working with SLP tokens, it is a normal part of the workflow for wallets to retrieve the Genesis data for the token. This is where critical metadata such as the `decimals` is stored. In the vast majority of cases, retrieving the `token_document_hash` data does not require any extra steps.

The BCH blockchain should be queried for the transaction data corresponding to the TXID stored in the `token_document_hash`field, and the OP_RETURN in that transaction should be decoded to retrieve the JSON data containing the MSP address.

### 4.2 Retrieve the Token State

The transaction history of the MSP address should be retrieved, and the list of TXIDs should be ordered with respect to block height. Newest transactions should be evaluated first.

With the ordered list of TXIDs, the wallet can walk the history of the MSP address and retrieve the first valid entry as the current state of the token. A valid entry meets the following requirements:

- The OP_RETURN of the transaction contains an IPFS hash, preferably following the [MSP specification](./ps001-media-sharing.md).
- The first input of the transaction originates from the MSP address. This ensures the holder of the private key updated the state, and the state was not 'spoofed' by another BCH address.

Once the first valid state data is found, the search can be ended and the wallet can act on the state. Other entries act as an immutible, append-only log of previous state. This is useful for other applications, but not necessary for a wallet to act on the current state.

### 4.3 Retrieve State from IPFS

With the IPFS hash, the wallet can then retrieve the state of the token from any IPFS gateway or any node on the IPFS network. It is assumed this data is also in JSON format.
