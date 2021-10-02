# Multisig Minting

## Permissionless Software Foundation Specification 005 (PS005)

### Specification version: 1.0.0

### Date originally published: October 2, 2021

### Date last updated: October 2, 2021

## Authors

Chris Troutner

## 1. Introduction

This is just a sketch to get the idea out. This will be refined in the future. Below if a _very_ rough draft of a workflow for minting new tokens. As opposed to using the low-level blockchain primitives, the multisig feature is instead done at a higher level in a node.js app. Several secrets are combined to decrypt a 12-word mnemonic that controls the minting baton.

Shamir's Secret Sharing
https://dev.to/simbo1905/shamir-s-secret-sharing-scheme-in-javascript-2o3g

### Brainstorming multisignature minting program.

Overview:

- A JavaScript program that would allow n-of-m signers to mint new tokens.
- The 'minting address' is the address holding the minting baton.
- The message they each sign would be an instruction on how many tokens to mint and where to send them.
- OP_RETURN Message components:
  - MINT header.
  - _qty_ of tokens to mint
  - _addr_ the address to send them to.
  - _nonce_ all signers must include the same nonce, to uniquely identify the round of minting.
  - _secret key_ Shamir's secret key, as a hex string
- The message encrypted with Shamir's secret key is the 12-word mnemonic of the minting address.
  - The encrypted 12-word mnemonic is hard-coded into the app.
- The OP_RETURN message is encrypted with the public key of the minting address.
  - The encrypted message is published to the blockchain in an OP_RETURN message.
  - The minting address is tagged with some dust.
- The node.js program for minting new tokens is stateless. It can be published to IPFS and stored on Filecoin.
  - Anyone can run the program at any time. It should only execute once.
- The address holding the minting baton is public and part of the program. Known by the minters.
- Commands needed: mint, move (baton), invalidate (minting round)
- The app marks a nonce as invalid after a mint succeeds.

### Basic workflow:

- Setup:

  - 'm' signers are each given a key from Shamir's Secret Sharing algorithm. This is a string in hex format.
  - To mint new tokens, each signer is given a nonce. An integer number that has not been used before.
  - A minimum of 'n' signers broadcast a transaction with an OP_RETURN, and a dust output sent to the minting address.
  - The OP_RETURN message is encrypted with the public key of the minting address.
  - Example of an unencrypted message: 'MINT qty addr nonce secret-key'

- Immutable app:
  - Download the transaction history of the minting address.
  - Decrypt each message with an OP_RETURN, and add them to an array.
  - Identify all the nonces that are invalid.
  - Collect the messages with the highest nonce value.
  - Collect the secret keys.
  - Drypt the mnemonic
  - Mint new tokens and send them to the specified address.
  - Broadcast an OP_RETURN transaction to mark the nonce of the current round invalid.

### Next steps:

- Create a repository:
  - A directory with a fork of the memo-push tool, for publishing messages with OP_RETURNs.
  - Use interactive prompt to collect the info.
  - A copy of the public program.
  - An app for generating the setup information.
