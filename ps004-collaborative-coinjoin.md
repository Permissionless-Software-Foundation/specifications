# Collaborabive CoinJoin

## Permissionless Software Foundation Specification 004 (PS004)

### Specification version: 1.0.0

### Date originally published: September 22, 2020

### Date last updated: May 20, 2021

## Authors

Chris Troutner

## 1. Introduction

[CashShuffle](https://cashshuffle.com/) and [CashFusion](https://cashfusion.org/) are popular [CoinJoin](https://en.bitcoin.it/wiki/CoinJoin) protocols on the Bitcoin Cash network. But they are complex and as a result have only been implemented in a single wallet ([Electron Cash](https://electroncash.org/)). A more general-purpose approach to generating CoinJoin transactions and coordinating wallets is desirable in order to allow wild innovation and integration into many wallets. This document describes the orchestration of a handful of JavaScript npm libraries that can be used to create that general-purpose approach.

## 2. Component Parts

On the surface, a CoinJoin transaction is not a difficult or unusual type of Bitcoin transaction. The hard part is the coordination and communication of wallets, as several wallets need to coordinate synchronously in order to generate and broadcast a CoinJoin transaction.

The Collaborative CoinJoin protocol specified in this document is based on [this CoinJoin example](https://github.com/Permissionless-Software-Foundation/bch-js-examples/tree/master/applications/collaborate/coinjoin). However, examples of how to generate a collaborative CoinJoin transaction is only one piece of the solution. A way to securely communicate, and a protocol for coordination, is also required.

For secure communication, the [bch-encrypt-lib](https://github.com/Permissionless-Software-Foundation/bch-encrypt-lib) can be used. This is a JavaScript npm library that contains utility functions for end-to-end (e2e) encryption of messages using the same elliptic curve encryption that Bitcoin is based upon.

For coordination the [IPFS](https://ipfs.io) network will be used, specifically the pubsub features that allow IPFS nodes to subscribe to channels and receive published messages. The [ipfs-coord](https://github.com/Permissionless-Software-Foundation/ipfs-coord) library was specifically designed for the type of coordination needed for this application. A demo of the library can be interacted with at [chat.fullstack.cash](https://chat.fullstack.cash), and the technology behind it is documented in a [series of blog posts and YouTube videos](https://troutsblog.com/blog/). This provides the foundation for the coordination and communication required to organize CoinJoin transactions between wallets.

## 3. Base Coordination

To bootstrap coordination between peers (wallets), a base pubsub channel needs to be established. The [PS001 Media Sharing protocol](https://github.com/Permissionless-Software-Foundation/specifications/blob/master/ps001-media-sharing.md) or more basic [Memo.cash protocol](https://memo.cash/protocol) provides an excellent medium to establish this base pubsub channel.

A simple pubsub channel can be published to the BCH blockchain. The channel can be any arbitrary string, for example 'myCoinJoinChannel1234'. The important thing is that the particular channel can be detected by wallets who want to coordinate. Writing messages to the uncensorable blockchain is a perfect way to communicate the channel name to wallet nodes.

## 4. Peer Coordination

Once a wallet has subscribed to the IPFS pubsub channel for base coordination, they can periodically broadcast their connection information. This allows other wallets watching the channel to find one another.

It's important to choose a time period that is not too soon (spammy), but not too long (bad UX). A period of two (2) minutes is widely acceptible as a good time frame.

When broadcasting connection information, it's important that each wallet broadcast enough information to allow other wallets to communicate with them using e2e encryption, but not enough information is broadcast to compromise security. Here is a list of recommended data to include in the broadcast:

- A schema version
- A message type or description indicating this is an 'announcement' of existance.
- The IPFS peer ID for the wallet.
- A list of IPFS multiaddresses for connection directly between nodes.
- The Bitcoin Cash address for the wallet, used for e2e encrypted message passing.
- The SLP address generated from the above BCH address, used if any token staking or token-related information is needed.
- The public key used to generate the BCH and SLP addresses above. This public key will be used to encrypt messages for this wallet.

When announcing itself on the base coordination pubsub channel, the wallet should also create its own pubsub channel using its `bitcoincash:` address as the pubsub channel string. Other wallets can communicate with this wallet by sending encrypted messages to its pubsub channel.

**Security Best Practices**

If a message is sent to a nodes private channel that is not e2e encrypted, the message should be rejected. It is also recommended that wallets change their IPFS and BCH information after each successful CoinJoin transaction.

## 5. Soliciting for CoinJoin Participation

Wallets become aware of other wallets by monitoring the base coordination pubsub channel. When a new wallet announces itself, a wallet should record the announcement information. It should periodically poll the wallets that it's aware of with a request to join a CoinJoin transaction.

The wallet can poll other wallets by sending a encrypted petition to each wallet using their `bitcoincash:` pubsub channel. The wallet should specify the following information in this petition:

- Minimum number of participants acceptable for a CoinJoin transaction.
- Maximum amount of BCH output in a CoinJoin transaction.

Wallets can track the above requirements between other peer wallets. When the requirements are met, the wallets can then coordinate to generate a CoinJoin transaction.

- Each wallet updates an array of petitions as the data comes in, with the newly updated information. Old data is replaced by new data.
- After each petition is received and the state updated, the state is checked for:
  - more than the minimum participants
  - With at least one peer with a maximum amount equal to or less than the wallets maximum

## 6. Coordination Protocol

- If the two requirements above are met, the wallet compiles an object containing:

  - The participants it wants
  - The maximum output of the CoinJoin (based on the smallest maximum of the peers)
  - a UUID for the organization message

- The object is broadcast to the selected participants, e2e encrypted, on their private pubsub channels.

- The wallet will wait approximately 2 minutes for all the wallets to respond. Communication will go through a series of phases until a successful CoinJoin transaction is built and broadcasted.

### 6.1 TX Building

- The wallet will wait until all particpants have acknowledged and agreed to the transactions.

  - The acknowledgement message must include:
    - The UTXO inputs they want to add to the transaction.
    - The outputs they want to add to the transaction.

- The wallet will compile an unsigned transaction from the ack messages. It will then broadcast the unsigned tx to the participants.

- Each participant will check their part of the transaction. If acceptible, they will sign and send back the partially-signed transaction.

- The wallet will wait approximately 2 minutes for all the wallets to respond.

### 6.2 Broadcasting

- Once all participants have responded with their partially signed transaction, the wallet will combine them into a single fully-signed tx and will broadcast it to the network.

### 6.3 Done

- The wallet will send each participant a _Done_ message, containing the txid of the transaction.

### 6.4 Canceling

- At any time, a wallet can cancel the transaction by sending a Cancelation message containing the UUID for Organization message.

### 6.5 Corner Cases

- If a wallet receives a new _Organization_ message before it receives a response from all participants, it should send out a _Canceling_ message to all participants and then respond to the _Organization_ message. This ensure that all participants default into collaborative behavior instead of selfish behavior.

## 7 Cleanup

After receiving a _Done_ message, all participants should change their IPFS and BCH connection information. This ensures that malicious participants can not track and associate transactions with nodes beyond the single transaction.
