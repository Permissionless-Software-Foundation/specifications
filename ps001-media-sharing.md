# Media Sharing Protocol Specification
## Permissionless Software Foundation Specification 001 (PS001)

### Specification version: 1.0.2
### Date originally published: May 30, 2020
### Date last updated: December 27, 2020

## Authors
Chris Troutner

## Acknowledgements
- James Cramer created the [Bitcoin Files Specification](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md) leveraged by this document.
- The [Memo Protocol](https://memo.cash/protocol) used by [Memo.cash](https://memo.cash) is used here as the base protocol.

## 1. Introduction
The following presents a simple protocol for sharing files both on-chain and off-chain with people using a Bitcoin Cash address as an identity. This specification defines a system that functions very much like email, using Bitcoin Cash addresses in place of an email address. It defines how to pass arbitrary messages and files of any size to the recipient. This content can be end-to-end (e2e) encrypted with the recipients public key, which means only the person holding the private key for that Bitcoin Cash address can decrypt the content.

For very small content (less than 10kB), on-chain data can be transmitted via the [Bitcoin Files Protocol](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md). Content of any size can be shared off-chain via the [Inter-Planetary File System](https://ipfs.io) (IPFS).

## 2. Protocol

This protocol specification describes the requirements for signaling to a Bitcoin Cash address that another address has sent it a message and how to retrieve the message.

### 2.1 Payload Metadata
The OP_RETURN of a Bitcoin Cash transaction is used to point to the message payload, and to indicate if that content is stored on-chain or off-chain. This data extends the [Memo Protocol](https://memo.cash/protocol), by using the `0x6dd2` prefix to signal a UTF-8 encoded (aka 'clear text') message using the memo protocol. (See [Appendix 2](#appendix-2) for more information.)

The message follows this pattern:

- `MSG <medium> <pointer> <subject>`

For example, here is a message indicating an off-chain message using IPFS as the content-delivery medium:

- `MSG IPFS QmT17Px3WcydqbZnKGUkKb5tWTM7Ypoz1UJ1MHWngC49xQ A message for you`

The *medium* above is specified as IPFS. The *pointer* is the IPFS hash needed to retrieve the content from the IPFS network. The rest of the signal is a *subject* message, similar to an email subject. The subject can be of any length, as long as it fits within the constraints of the OP_RETURN.

Here is another example indicating an on-chain message using the [Bitcoin Files Protocol](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md) protocol:

- `MSG BFP bitcoinfile:2f2add68a365da4cae325e4d4a0f6a57ddffd3446a6dc3bf5b32f6ae9f0f48ff Another subject message`

### 2.2 Payload Content
The payload content should be a simple [JSON file](https://en.wikipedia.org/wiki/JSON) containing a combination of plain-text and encrypted information. Arbitrary additional fields can be added, but at a minimum, the JSON file should contain the following properties:

- **receivers** - an array of Bitcoin Cash addresses that are the intended recipient of the message.
- **sender** - A single Bitcoin Cash address indicating the sender of the message.
- **subject** - A copy of the clear-text subject encoded in the OP_RETURN.
- **message** - An array of data containing the message payload content.
  - If not encrypted (clear-text), the array only requires a single element, which contains the clear-text content of the message.
  - If encrypted, each element of the array should contain a copy of the hex-encoded encrypted message data. The encrypted data in one element should be encrypted with the public key of the address in the same element location in the *receivers* array. e.g. message[2] should be encrypted with the public key of the address in receivers[2].


Here is an example of a simple message sent with [message.FullStack.cash](https://message.fullstack.cash):

```
{
  "receivers": ["bitcoincash:qzjgc7cz99hyh98yp4y6z5j40uwnd78fw5lx2m4k9t"],
  "sender": "bitcoincash:qqlrzp23w08434twmvr4fxw672whkjy0py26r63g3d",
  "subject": "Re: A Test Message",
  "message": ["0478ad067871eea98b1ded073eeb8c48dd808e556d09edaa6066e1b7c386d1cbb6cfed0082df6d5510e077f272c693729b159bf37542b0192e39f73a8d71f6f96afc48ba8dc1477d579fe749088bddee4155bd462d93f8eff4bc3e416938dfe71f8ed01eb440f81b39760a9d874c640da4085db9aa50bb4471bfbfd45501a98fcd91f6e3f3271c92db12e96e8ff40ad53b551cdb48bc7399be281db183fec2f7427546291cc6dd31d86ad732c363218f7e4a4d567f3a9fb36e011e3c2c7fe9b7cdee66438c0d967d66de2c851f83e44c1e2602e73af4648334dbb5f2d519fd3553e8ceed782a790bcd7e8f9e5e99f14bd88cd777bc1eaef244e56b0e99cfd8f7cff917576c87b802ec528392f1be83d11601253daac1d6d9c7f5dbc273c93dd8961560af6ae5776384eb5d962508033b01a41ffe536e22723b596742bc1b3d66247b204d030e1faff225d7bfed38922918d69b2269e350b41d150f2e6ebbb3a0911086cc1e7b1f0a6db5ab97de5cda82ff"]
}
```


### 2.3 Message Signaling

An on-chain signal is required to allow wallets to detect that they have a message waiting for them. In addition to the data in the OP_RETURN, a dust output is sent to the recipients address. This will cause the transaction to appear in the transaction history for the address. Wallets can easily crawl their transaction history to find transactions with the above OP_RETURN payload.

- [Here is an example of a transaction on the block explorer](https://explorer.bitcoin.com/bch/tx/65395fe21e1add6bfb249f6ad108834734b0f71379b67a222f4144cbb39dfa32). The recipient is bitcoincash:qzxk8ecxm6drkcjtkrepesx5dd45fsvjauvxeeynfy.

### 2.4 Spam Prevention

The on-chain signal to the recipient is a standard dust output (546 satoshis). But it is not required to be dust. It can be any amount of satoshis more than dust.

If a threshold amount of satoshis is set by the end user, this feature can be used as spam prevention. Any messages not meeting the threshold can be ignored by the wallet.

### 2.5 Multiple Parties

Multiple parties can be signaled with a single transaction by appending additional outputs to the transaction. This will ensure the transaction appears in the transaction history of each address involved.

If encrypted, the payload should contain a copy of the same message for each party, encrypted with that parties public key.

This variation captures the following use cases:
- Sending a message to multiple parties at once.
- Ensuring the sender of a message can view a 'sent' copy of the e2e encrypted message.

## 3. Encryption

The protocol can be used with or without encryption. However, the Bitcoin protocol naturally allows for end-to-end (e2e) encryption to be implemented, far easier than is currently done in email. Bitcoin payments naturally use Eliptic Curve cryptography. This aspect of the Bitcoin protocol can be leveraged to encrypt the content of messages as well.

### 3.1 Public Keys

In email, recipients must provide the sender with their public key. The sender then encrypts the message with that public key before sending the message. This key passing is a large source of friction and largely explains why attempts to implement widespread encryption in email has been a failed effort. Key servers are available to reduce this friction, but they require additional training by the users.

Any time a Bitcoin Cash address **sends** money, their public key is recorded on the blockchain. This is a core, primitive part of the Bitcoin protocol. This key can be retrieved by the sender *without any need for communication with the recipient*. This eliminates the source of friction experienced in the email use-case.

However, unless the recipient has initiated **at least one transaction**, their public key will not exist on the blockchain and it can not be retrieved. This is an important initialization step that wallets implementing this specification need to be aware of.

## 4. Implementation
There have been multiple implementations of this specification as this idea has evolved over time. The list below are presented in reverse-chronological order, so newest first.

- [encrypt-msg](https://github.com/Permissionless-Software-Foundation/encrypt-msg) is a command line application that implements the full specification in this document.
- [memo-push](https://github.com/christroutner/memo-push) is another command line application that can write IPFS hashes to the BCH blockchain.
- [Uncensorable Publishing](https://uncensorablepublishing.com/) was an early implementation of this specification for publishing websites. It leverages IPFS and the memo-push command line app.

## Appendix 1
### Message Encryption and Delivery Workflow

- [Dia diagram file](./images/e2e-pt2-flow.dia)

This diagram shows the workflow associated with 'Alice' to send an e2e encrypted message to 'Bob' using the encrypt-msg app. This diagram is covered in detail in [this YouTube video](https://www.youtube.com/watch?v=RB9yt65y9s8).

![e2e-pt2-flow.png](./images/e2e-pt2-flow.png)

## Appendix 2
### Extending the Existing Protocols

This specification uses the OP_RETURN prefix `0x6dd2`. This choice is a result of collaboration with the developers behind [memo.cash](https://memo.cash) and [member.cash](https://member.cash), two on-chain social media applications.

The `0x6dd2` prefix was previously unused, and it extends the existing specifcation for each social media platform:

- [memo.cash Protocol Specification](https://memo.cash/protocol)
- [member.cash Protocol Specification](https://github.com/memberapp/protocol)
