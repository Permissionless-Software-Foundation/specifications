# Multisig Rate Approval (PS009 v2)

## Permissionless Software Foundation Specification 009, Revision 2 (PS009-R2)

### Specification version: 2.0.0 (draft, added with PS010 v2.0.0)

### Date originally published: January 29, 2023

### Date last updated: November 27, 2026

### Supersedes

This revision supersedes **PS009 v1 — Multisig Approval**:

- Live v1.0.2 document: [ps009-multisig-approval.md](ps009-multisig-approval.md)
- Archived v1.0.2 (permalink at the pre-revision master commit,
  `3ec3699`):
  https://github.com/Permissionless-Software-Foundation/specifications/blob/3ec369935b2d11bd551ba542c104a84bb6641e23/ps009-multisig-approval.md

PS009 v2 uses the approval-transaction workflow: an Update Transaction
approved by a P2SH multisig Approval Transaction, with council membership
keyed to SLP NFTs. Re-scoped for the File Pinning Protocol (PS010): the
council is this network's node-operators (not the PSF Minting Council), and
the approved datum is the on-chain `PSP1` registry record — the `satsPerMb`
rate and pool payout address. There is **no PSF token anywhere in this
revision**; the SLP NFT appears only as an operator-identity beacon and a
carrier for per-operator performance data.

## Authors

Chris Troutner (v1); v2 revision drafted for the File Pinning Protocol
(PS010) under the PSF specifications process.

## 1. Introduction

This specification replaces PS009 v1 as the governance mechanism for the
[File Pinning Protocol (PS010)](ps010-file-pinning-protocol.md). Its single
job is to let a **group of node-operators — the registry council — jointly
approve the `satsPerMb` price and the pool payout address** for each registry
epoch (PS010 section 4). No single operator, and no external minting body, can
set the price or redirect the pool.

Council membership is proven by **holding an SLP NFT** — the proven, practical
way to discover and verify the approving parties on chain (section 2). The
Group Token is issued by the **registry council of node-operators** (not the
PSF Minting Council), with **one NFT per operator**. The approved object is
the PS010 registry epoch datum: the on-chain `PSP1` record carrying the
`satsPerMb` rate and pool payout address (PS010 section 4). PSF tokens play no
role: no PSF-token payments, no PSF-token governance, no PSF write-price
feed.

## 2. Council membership

The registry council is a set of node-operators of the PS010 network — the
people running a full Cash Stack (BCH full node + indexer + pin-service).
Membership is proven **by holding an SLP NFT under the council's Group
Token**:

- **One NFT per operator.** The Group Token is issued by the council itself;
  it is not a PSF token and is not linked to PSF governance.
- **Identity beacon.** The NFT is the stable handle: anyone can look up, on
  chain, the address and public key that currently hold the NFT. This is far
  easier in practice than having each member hand over a public key, and it
  means an operator's address and public key can change over time — the NFT
  moves to a new address while remaining the same operator's beacon.
- **Performance data.** The SLP token's mutable data carries each operator's
  performance reports (as recorded by the network, e.g. pin-service
  metering). This is how the network tracks each operator's reporting; it is
  not an input to approval validation.
- **Multisig construction.** The council's P2SH multisig address is built from
  the current NFT-holder public keys, following the procedure in the
  [psf-multisig-approval](https://www.npmjs.com/package/psf-multisig-approval)
  JavaScript library, which is expected to be **forked and adapted** — it is
  the implementation home for this workflow.
- **N-of-M threshold:** chosen by the council and published in the registry
  policy (e.g. 3-of-5). Public keys are compressed and sorted
  lexicographically before address derivation, so that any party recomputes
  the same address.
- **Not the PSF Minting Council:** this is a new council of node-operators,
  deliberately separate from the Minting Council and from any PSF-token
  governance.

Membership changes (join/leave, threshold changes, key rotation) are registry
policy, not protocol; they are recorded as new registry epochs / council
lookups by PS010 (section 4). Key compromise and expulsion are handled the
same way PS010 handles controller compromise (PS010 section 8).

## 3. Overview — two transactions

The workflow is unchanged in shape from PS009 v1, with one important scoping
change: **the approved datum lives on chain, not in IPFS.**

1. An **Update Transaction** is broadcast from a normal P2PKH address. Its
   OP_RETURN carries an IPFS CID resolving to a JSON document containing the
   **council membership proof** (who is approving) and a reference to the
   **on-chain `PSP1` announcement being approved** (PS010 section 4).
2. An **Approval Transaction** is generated from the council's P2SH multisig
   wallet. Its OP_RETURN carries the word `APPROVAL` and the TXID of the
   Update Transaction.

The datum being approved — the `satsPerMb` rate, the pool payout address and
`validFromHeight` — is the `PSP1` announcement record itself, stored
**directly on chain** (PS010 section 4.1). The IPFS JSON carries only the
membership proof. No IPFS lookup is needed to determine the rate or pool in
force at any height.

Any entity can walk the chain: PSP1 announcement → Update TX → CID → JSON
membership proof → recomputed multisig address → Approval TX spending from it.

## 4. Update Transaction

The Update Transaction is a standard P2PKH transaction whose first output is
an OP_RETURN containing an IPFS CID (utf-8). The CID resolves to a JSON
document carrying the following properties:

1. The token ID of the council's Group Token (the token used to generate the
   operators' NFTs).
2. The token IDs of each operator NFT, together with the address and public
   key holding each NFT at the time of the transaction.
3. The P2SH multisig address derived from those public keys, and the n-of-m
   threshold.
4. The **TXID of the on-chain `PSP1` registry announcement this Update
   endorses**. The `satsPerMb`, pool payout address `A_i` and
   `validFromHeight` of that announcement ARE the approved datum (PS010
   section 4.1, 4.4) — the chain is authoritative, never this JSON.

The first three properties validate the approval (who is approving); the
fourth binds the approval to a specific on-chain epoch record (what is being
approved). Each transaction validates the other.

### 4.1 Update Transaction example

- `OP_RETURN { "cid": "bafybei...redacted...", "endorses": "<PSP1 announcement txid>", "ts": 1770000000000 }`

## 5. Approval Transaction

The Approval Transaction must contain two pieces of information in its
OP_RETURN data:

1. The word `APPROVAL` — makes the transaction easy to parse and identify.
2. The TXID of the Update Transaction.

The Approval Transaction must include a dust output (546 satoshis) in its
second output (index 1) to a **Reference Address** — any known BCH address
agreed by all parties, used as the anchor for validation (PS010 section 4.2
consults the same Reference Address convention).

### 5.1 Approval Transaction example

- `OP_RETURN APPROVAL 6eaa12d2749cf15b00873551088975dd6748d9a6f867426db914735c58ad730a`

## 6. Validation

A PS010 node (or any interested party) validates an approved registry epoch as
follows:

1. Read the `PSP1` announcement under consideration directly from the chain
   (PS010 section 4.1 decode); note its txid, `satsPerMb`, pool payout
   address and `validFromHeight`. These are the epoch's operative values; no
   IPFS lookup is involved.
2. Request the transaction history for the Reference Address, sorted
   descending by block height. Find the most recent transaction containing
   `APPROVAL` in its OP_RETURN — the latest Approval Transaction — and note
   the address which generated it (the multisig address).
3. Retrieve the Update Transaction from the TXID in the Approval
   Transaction's OP_RETURN, and the CID from the Update Transaction's
   OP_RETURN.
4. Fetch and parse the JSON document for that CID from an IPFS gateway.
5. Verify the Group Token ID is listed and matches the council's expected
   Group Token.
6. Verify that all operator NFTs are children of that Group Token.
7. Verify that each address listed in the JSON is generated from the given
   public key.
8. Verify that each address holds the NFT listed. Since NFTs can move, it may
   be necessary to verify the address *held* the NFT at the relevant point in
   its transaction history — this is what makes the beacon robust to changed
   addresses and rotated keys.
9. Compute the P2SH multisig address from the public keys and the published
   threshold, assert it equals the address that generated the Approval
   Transaction, and assert the Approval Transaction's first input spends from
   that multisig address (i.e. it was signed by the n-of-m council keys).
10. Assert the JSON's `endorses` field equals the PSP1 announcement's txid
    from step 1, and that the announcement's first vin spends from the
    council's P2SH address (PS010 section 4.2). A mismatch is PS010 failure
    case F9 — the epoch is not admitted.

The operator performance reports carried in the SLP mutable data are read for
network bookkeeping only; they are not inputs to approval validation.

## 7. Relationship to PS009 v1

PS009 v1 — Multisig Approval — introduced the approval-transaction workflow
and remains valid for the systems already using it. This revision uses the
same workflow for PS010, re-scoped: membership is the registry council of
node-operators (not the PSF Minting Council), there is no PSF token, and the
approved datum is the on-chain `PSP1` registry record. PS010 section 8.2's
removal of *legacy PS009 token write-price lookups* refers to v1's data feed,
not to this approval workflow.

## 8. Status

Draft — added alongside PS010 v2.0.0 (PR #11) for review and is **not yet
normative**. The library implementation
([psf-multisig-approval](https://www.npmjs.com/package/psf-multisig-approval))
is expected to be revised in lockstep if this draft is adopted.
