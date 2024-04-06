# File Pinning Protocol

## Permissionless Software Foundation Specification 010 (PS010)

### Specification version: 1.0.0

### Date originally published: April 6, 2024

### Date last updated: April 6, 2024

## Authors

Chris Troutner

## 1. Introduction

This specification describes a protocol for storing (pinning) files on the [IPFS](https://ipfs.io) network. It is known as the Permissionless Software Foundation File Pinning Protocol (**PSFFPP**). This file hosting service can be paid through the use of [SLP tokens](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md). The protocol in this document describes a process for paying for the hosting of files (using the blockchain) to achieve a fully-decentralized process that is anonymous, permissionless, and censorship-resistant.

This specification builds on top of the [PS008 specification for Claims](https://github.com/Permissionless-Software-Foundation/specifications/blob/master/ps008-claims.md). This protocol centers around a **Pin Claim** which indicates files that should be pinned by the network, and also provides proof of payment (Proof of Burn). This specification is implemented in the [psffpp npm JavaScript library](https://www.npmjs.com/package/psffpp)

## 2. Transactions

The PSFFPP depends on two blockchain transactions:
- Proof of Burn (**PoB**) - proves that payment has been made by burning tokens.
- Pin Claim - indicates the file that should be pinned by the network, and contains the PoB transaction ID.


### Proof of Burn

A PoB transaction can encompass any SLP token, but this protocol focuses on the [PSF token](https://token.fullstack.cash/?tokenid=38e97c5d7d3585a2cbf3f9580c82ca33985f9cb0845d4dcce220cb709f9538b0). [An example of a PoB transaction can be viewed here.](https://token.fullstack.cash/transactions/?txid=307610a97f5a6e8c229679f380aa25eca65db7a1ac8bc5c236479d7406f7b81e) It is simply a [SLP SEND transaction](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md#send---spend-transaction) where the output is less than the input. The difference is considered 'burned'.

In the example transaction above, there were 7.57514589 as input, and 7.49179356 output, with a difference burned of 0.08335233 tokens. At the time of the transaction, that was the cost of writing up to 1MB to the PSFFPP network.

The cost (in tokens) for pinning data to the PSFFPP network is set by the [PSF Minting Council](https://psfoundation.info/governance/minting-council). They periodically set the price to target $0.01 USD per megabyte for pinning costs. The token price is loosely pegged to the Bitcoin Cash cryptocurrency, so fluctuates over time relative to the US dollar. As a result, the price is never precisely $0.01 USD.

A minimum cost of $0.01 USD in PSF tokens is required to pin content, even if that content is less than 1 MB in size. Files up to 100 MB can be pinned by the PSFFPP network. To find the current cost per MB of pinning file data, the [`getMcWritePrice()` function](https://github.com/Permissionless-Software-Foundation/psffpp?tab=readme-ov-file#get-the-write-price) can be called. That call will search the BCH blockchain for the latest price authorized by the PSF Minting Council.

### Pin Claim

A pin claim is a normal Bitcoin transaction that contains an OP_RETURN in the first output. The data in the OP_RETURN output has four parts:

1. `0x00510000` - Hexidecimal [Lokad ID](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/spec/op_return-prefix-guideline.md)
2. PoB TXID - The transaction ID for the Proof of Burn in binary format.
3. CID - The IPFS Content ID (CID) for the file to be pinned, in utf-8 format.
4. Filename - The original filename and extension of the file, in utf-8 format.

[Here is an example of a valid Pin Claim transaction](https://explorer.bitcoinunlimited.info/tx/0c952f7e47daa3d3bb62932c22f184f892ad189a428a3279c6c5968ed3e65819).

Pin Claims are tracked by the [pin-ipfs branch of the psf-slp-indexer](https://github.com/Permissionless-Software-Foundation/psf-slp-indexer/tree/pin-ipfs). When a Pin Claim transaction is detected by the indexer, the data is passed to the [ipfs-file-pin-service](https://github.com/Permissionless-Software-Foundation/ipfs-file-pin-service) software via a webhook. ipfs-file-pin-service will evaluate the Pin Claim and PoB transactions. If they meet the criteria above, and the PoB provides enough payment, the file is pinned by the software.

There are multiple instances of the ipfs-file-pin-service software running in the world. Each one independently evaluates and validates the Pin Claims they find, and they will indepenently pin the file content.

## 3. Supplimentary Information

### Documentation

- [PSFFPP.com](https://psffpp.com/docs/intro) - Official documentation site.

### Videos

- [upload.psfoundation.info Demo](https://youtu.be/d9AGMTRM3Ws?si=pNZkDcikPQO1Jpbe)
- [PSFFPP Technical Introduction](https://youtu.be/flaEm4RFzYA?si=adnqps_BKhzx0xy0)
- [psffpp-client](https://youtu.be/viX_SBpEgEU?si=sQCfDTOhyX04-l4H)

### Software

- [psffpp](https://www.npmjs.com/package/psffpp) - npm JavaScript library implementing this specification.
- [psf-slp-indexer, pin-ipfs branch](https://github.com/Permissionless-Software-Foundation/psf-slp-indexer/tree/pin-ipfs) used to track Pin Claims on the blockchain.
- [ipfs-file-pin-service](https://github.com/Permissionless-Software-Foundation/ipfs-file-pin-service) validates Pin Claims and pins IPFS files. Works in tandem with psf-slp-indexer.
