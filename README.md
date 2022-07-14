# Specifications
This repository is used to track PSF Specification documents. These are specifications and best practices used by the PSF community.

- [PS001 - Media Sharing Protocol](./ps001-media-sharing.md) - A simple protocol for sharing files both on-chain and off-chain with people using a Bitcoin Cash address as an identity. This specification defines a system that functions very much like email, using Bitcoin Cash addresses in place of an email address. It defines how to pass arbitrary messages and files of any size to the recipient. This content can be end-to-end (e2e) encrypted with the recipients public key, which means only the person holding the private key for that Bitcoin Cash address can decrypt the content.

- [PS002 - SLP Mutable Data](./ps002-slp-mutable-data.md) - How to leverage the `document hash` field when creating an SLP token to point to mutable data (data that can change over time).

- [PS003 - SLP JSON ID](./ps003-slp-json-id.md) - Inspired by the [Lokad ID](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/spec/op_return-prefix-guideline.md), anyone can submit a PR to claim their `slpjsonid`. Wallets can download the [slpjsonid.csv file](./slpjsonid.csv) to decipher the JSON data uploaded via [PS002](./ps002-slp-mutable-data.md).

- [PS004 - Collaborative CoinJoin](./ps004-collaborative-coinjoin.md) - A [CoinJoin protocol](https://en.bitcoin.it/wiki/CoinJoin) for achieving financial privacy when using Bitcoin **Cash**. The 'Cash' label implies that it is anonymous like physical cash is. Collaborative CoinJoin helps wallets achieve financial privacy without any central coordination server. It results in large, anonymous coins that do not suffer from [the 'dust' problem](https://academy.binance.com/en/articles/what-is-a-dusting-attack).

- [PS006 - Simple Store Protocol (SSP)](./psf006=simple-store-protocol.md) - A protocol for creating Stores and Products that are difficult to censor.
