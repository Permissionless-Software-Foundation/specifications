# SLP JSON ID
## Permissionless Software Foundation Specification 003 (PS003)

### Specification version: 1.0.0
### Date originally published: July 27, 2020
### Date last updated: July 27, 2020

## Authors
Chris Troutner

## Acknowledgements
- This specification uses the [Loken ID specification](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/spec/op_return-prefix-guideline.md) as a template for the way a list of IDs will be maintained.
- This specification is largely an extension of the [PS001 Media Sharing Protocol](./ps001-media-sharing.md) and the [PS002 SLP Token Mutable Data Specification](./ps002-slp-mutable-data.md)
- [Joaness Vermorel](https://blog.vermorel.com/journal/2018/5/23/4-byte-prefix-guideline-for-op_return-on-bitcoin.html) created the Lokad ID specification that this specification was inspired by.

## 1. Introduction
The [PS002 specification](./ps002-slp-mutable-data.md) describes a way of uploading arbitrary JSON data to the Bitcoin Cash blockchain and attach a point to that data to a new SLP token. This specification builds on top of that one by specifying a common field (`slpjsonid`) that can be used to help wallet software locate schema data for automatically understanding the data in the JSON file.

The [Lokad ID specification](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/spec/op_return-prefix-guideline.md) describes a method for updating [an open source CSV file](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/etc/protocols.csv). Wallet software that downloads this CSV can use it to easily identify the different protocols making use of the OP_RETURN. In this same way, [slpjsonid.csv](./slpjsonid.csv) is an open source CSV file that wallets can use to decipher the JSON data uploaded to the BCH blockchain via the PS002 specification.
