# Specifications
This repository is used to track PSF Specification documents. These are specifications and best practices used by the PSF community.

- [PS001 - Media Sharing Protocol](./ps001-media-sharing.md) - A simple protocol for sharing files both on-chain and off-chain with people using a Bitcoin Cash address as an identity. This specification defines a system that functions very much like email, using Bitcoin Cash addresses in place of an email address. It defines how to pass arbitrary messages and files of any size to the recipient. This content can be end-to-end (e2e) encrypted with the recipients public key, which means only the person holding the private key for that Bitcoin Cash address can decrypt the content.

- [PS002 - SLP Mutable Data](./ps002-slp-mutable-data.md) - How to leverage the `document hash` field when creating an SLP token to point to mutable data (data that can change over time).

- [PS004 - Collaborative CoinJoin](./ps004-collaborative-coinjoin.md) - A [CoinJoin protocol](https://en.bitcoin.it/wiki/CoinJoin) for achieving financial privacy when using Bitcoin **Cash**. The 'Cash' label implies that it is anonymous like physical cash is. Collaborative CoinJoin helps wallets achieve financial privacy without any central coordination server. It results in large, anonymous coins that do not suffer from [the 'dust' problem](https://academy.binance.com/en/articles/what-is-a-dusting-attack).

- [PS006 - Simple Store Protocol (SSP)](./ps006-simple-store-protocol.md) - A protocol for representing real-world Stores (like a grocery store) as a token on the blockchain. This protocol is used by [LocalTradeList.com](http://localtradelist.com).

- [PS007 - Token Data Schema](./ps007-token-data-schema.md) - A guide to creating immutable and mutable data for tokens.

- [PS008 - Claims]('./ps008-claims.md') - Is used to make on-chain claims about Tokens or other Claims. This is a way for user to leave on-chain comments about tokens, and compliments the Simple Store Protocol (PS006).

- [PS009 - Multisig Approval](./ps009-multisig-approval.md) - Describes how a group of NFT holders can generate an on-chain 'stamp of approval' for arbitrary data. This can be used to securely distribute data to a decentralized network. The cononical use-case is the [PSF Minting Council](https://psfoundation.info/governance/minting-council) setting the price of writes to the [P2WDB](https://p2wdb.com).

- [PS010 - File Pinning Protocl](./ps010-file-pinning-protocol.md) - Introduces the PSF File Pinning Protocol (PSFFPP) implemented at [explorer.psffpp.com](https://explorer.psffpp.com) and described at [psffpp.com](psffpp.com). This is a protocol using SLP token burning to indirectly pay for the cost of hosting files on a decentralized IPFS network of nodes.

- [PS011 - User Data Schema & Harmonization](./ps011-user-data-schema-harmonization.md) - Describes the schema used in the `userData` section of the Mutable Data of an SLP token, across [Token Tiger](https://tokentiger.com), [SLP DEX](https://dex.psfoundation.info), and future apps. This allows 'harmonization' for cross-capability usage of tokens.
