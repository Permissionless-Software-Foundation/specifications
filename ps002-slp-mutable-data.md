# SLP Token Mutable Data

## Permissionless Software Foundation Specification 002 (PS002)

### Specification version: 1.2.0

### Date originally published: July 20, 2020

### Date last updated: October 14, 2022

## Authors

Chris Troutner

## Acknowledgments

- This document builds on top of the [SLP Token Type 1 Protocol](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md).

## 1. Introduction

This specification has been implemented in the [slp-mutable-data](https://www.npmjs.com/package/slp-mutable-data) npm library.

The Simple Ledger Protocol (SLP) for tokens includes a `token_document_hash` field in the [specification for the Genesis transaction](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md#genesis---token-genesis-transaction). This field is intended to hold a TXID for a Bitcoin Cash transaction. This TXID was originally intended to point to an on-chain file uploaded with the [Bitcoin Files Specification](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md), but can be used with any arbitrary BCH transaction.

This document specifies how to leverage the `token_document_hash` field when creating a SLP token, to point to mutable data (data that can change over time). This provides many new use cases. Here are a few examples:

- Token Icons
- Tracking state of tokenized assets (like a video game character or supply chain status)
- Security tokens compliant with arbitrary regulation rules.
- NFTs that leverage both mutable and immutable data.

Mutable data is controlled by a key pair. This specification is a general approach for:
- Token creators to add mutable data at the time of token creation.
- Wallets to discover mutable data associated with a token.

In this specification, [IPFS](https://ipfs.io) is used for off-chain data storage. It should be understood that the intention is for developers implementing this specification to upload that data to the Filecoin blockchain, and thereby make the data permanently available over IPFS. The easiest service to do this is [web3.storage](https://web3.storage).

## 2. Protocol Overview

There are four parts to _encoding a pointer_ to mutable data and attaching it to the Genesis transaction, when creating a new SLP token:

1. Initialize the *mutable data address*.
2. Create immutable data (optional).
3. Create the token.
4. Update mutable data.

Step two is optional. It allows the token creator to attach immutable (unchangeable) data to the token at the time of creation.

## 3. Initialize the Mutable Data Address

Mutable data is controlled by a key pair (An address and a private key). Whomever controls the private key can update the mutable data. The output of this step is a TXID which can be included in the tokens `token_document_hash` field.

- Generate a new private/public key pair for controlling the mutable data (the *mutable data address*).
- Broadcast a transaction (from any address) with the following properties:
  - The **first** output contains an OP_RETURN with a JSON containing an `mda` key with the value being the *mutable data address*.
    - Example: `{"mda": "bitcoincash:qrjptppjcqh3yvvrcxw5tat47rhvrladuvvkh6p5ed"}`
  - The **second** output contains an output going to the *mutable data address*.
- The TXID from this transaction goes into the `token_document_hash` field of the new token.

## 4. Create Immutable Data (Optional)

This step is optional. It writes a [IPFS CID](https://proto.school/anatomy-of-a-cid/01) to the tokens `token_document_url` field. This field is often used to display a URL associated with the token, which is a limited amount of data. Moving that data to a JSON file, linked to the token by a CID, allows for unlimited data, while retaining the immutable nature of the `token_document_url` field.

- Generate a JSON file containing any information that should be captured in the immutable data.
- Upload the JSON object to IPFS, which results in a CID.
- Write the CID into the `token_document_url` field of the token at the time of creation. Use the `ipfs://` prefix to explicitly indicate that it is an IPFS CID, like this:
  - `ipfs://QmTu6DWuAdq36EZQHGs1QDhSJBpbZYcpcbGH9S42tNB1aX`

## 5. Create the Token

There is no change to the workflow described in the [SLP Token Specification](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md) when creating a token. This specification simply leverages these two properties of the Gensis transaction:

- `token_document_hash` contains the TXID from section 3 for retrieving mutable data.
- `token_document_url` contains the CID from section 4 for retrieving immutable data.

## 6. Update Mutable Data

Updating mutable data should be executed at least once, immediately following the creation of the token, so that wallets can properly retrieve the data. However, the steps below can be executed at any time, to update the token to point to the latest data.

The *mutable data address* must have some BCH to pay transaction fees. Updates not originating from the *mutable data address* are ignored. This prevents malicious updates from other addresses. Only the owner of the *mutable data address* private key can update the mutable data.

- Generate a JSON file and upload it to IPFS. This results in a CID.
- Generate a transaction with the following properties:
  - The **first input** must be spent from the *mutable data address*.
  - The **first output** must be an OP_RETURN containing JSON with the following key-value pairs:
    - A key of `cid` and a value of the CID representing JSON data that can be retrieved over IPFS.
    - A key of `ts` (for *timestamp*) and a value containing a number representing a [JavaScript date](https://www.w3schools.com/jsref/jsref_gettime.asp).

Example:
```
{
  "cid": "ipfs://QmTu6DWuAdq36EZQHGs1QDhSJBpbZYcpcbGH9S42tNB1aX",
  "ts": 1665768143885
}
```

The timestamp is used to sort multiple entries within the same block. Block height is used to determine the most recent update of the mutable data. But if multiple entries make it into the same block, then the timestamp is used to sort the entries within that block. The most recent update is used as the current data.

## 7. Reading Mutable Data

Reading mutable data is the process by which blockchain software (like [bch-api](https://github.com/Permissionless-Software-Foundation/bch-api)) retrieves the mutable data associated with a token. The process is as follows:

- Given the `token ID` of the token, the Genesis data for the token is retrieved.
- Get transaction data for the TXID in the `token_document_hash` field.
- Get the *mutable data address* from the second output of the transaction data OR the OP_RETURN in the first output of the transaction.
- Get the transaction history for the *mutable data address*. It should be in chronological order with the newest TX first.
- Traverse the transaction history until a TX with the following properties are found:
  - First input is spent by the *mutable data address*.
  - First output is an OP_RETURN containing JSON with a `cid` property.
- Retrieve the JSON object from IPFS using the CID.

## 8. Mutable Data Recommendations

The mutable data can be any kind of arbitrary data, however [JSON-LD](https://json-ld.org/) formatted Linked Data following the [Schema.org](https://schema.org/) schema is recommended. Here is an example of what a token icon might look like:

```
{
  "tokenIcon": {
    "@context": "https://schema.org",
    "@type": "ImageObject",
    "author": "Chris Troutner",
    "contentUrl": "https://hub.textile.io/ipfs/bafkreiasjlveyusmkgludz6vopzg5t5g3lefgtu5oudoawjrcttmgwjea4",
    "datePublished": "2021-05-20",
    "description": "PSF Logo",
    "name": "psf-logo.png"
  }
}
```
