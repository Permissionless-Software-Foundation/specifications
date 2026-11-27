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

v2 keeps the v1 workflow (an Update Transaction approved by a P2SH multisig
Approval Transaction) but replaces the SLP/PSF-token membership model with a
plain BCH council of node-operators, and re-focuses the approved datum on the
File Pinning Protocol's `satsPerMb` rate.

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

v1 of this specification derived council membership from holders of SLP NFTs
belonging to a PSF Group Token, and approved PSFFPP *write-price* data
paid-for in PSF tokens. v2 removes both: membership is a multisignature wallet
built directly from the node-operators' BCH public keys, and the approved
object is the PS010 registry epoch datum (`satsPerMb` + pool address). There
is **no PSF token and no SLP dependency anywhere in this revision.**

## 2. Council membership

The registry council is a set of node-operators of the PS010 network — the
people running a full Cash Stack (BCH full node + indexer + pin-service).
Each operator contributes one standard BCH public key. The council's
multisignature wallet is built from those keys:

- **N-of-m threshold:** chosen by the council and published in the registry
  policy (e.g. 3-of-5).
- **Construction:** multi-signature P2SH (`OP_m ... OP_n OP_CHECKMULTISIG`),
  e.g. via the multi-hd / `createMultisig` facility of
  [psf-bch-wallet](https://www.npmjs.com/package/psf-bch-wallet). Public keys
  are compressed and sorted lexicographically before address derivation, so any
  party recomputes the same address.
- **No SLP Group Token, no NFTs, no PSF tokens.**
- **Not the PSF Minting Council:** this is a new council of node-operators,
  deliberately separate from the Minting Council and from any PSF-token
  governance.

Membership changes (join/leave, threshold changes, key rotation) are registry
policy, not protocol; they are recorded as new registry epochs / council
lookups by PS010 (section 4). Key compromise and expulsion are handled the
same way PS010 handles controller compromise (PS010 section 8).

## 3. Overview — two transactions

The workflow is unchanged in shape from PS009 v1:

1. An **Update Transaction** is broadcast from a normal P2PKH address. Its
   OP_RETURN carries an IPFS CID resolving to a JSON document containing the
   approved datum (`satsPerMb`, pool address) plus the council material needed
   to validate the approval.
2. An **Approval Transaction** is generated from the council's P2SH multisig
   wallet. Its OP_RETURN carries the word `APPROVAL` and the TXID of the
   Update Transaction.

Any entity can walk the chain: Update TX → CID → JSON → recomputed multisig
address → Approval TX spending from it. Each transaction validates the
other, exactly as in v1 (PS009 v1 section 5).

## 4. Update Transaction

The Update Transaction is a standard P2PKH transaction whose first output is
an OP_RETURN containing an IPFS CID (utf-8). The CID resolves to a JSON
document carrying the following properties:

1. The council P2SH multisig address and the compressed public keys used to
   derive it (the council's current member keys).
2. The n-of-m threshold.
3. The approved `satsPerMb` (uint64, satoshis per megabyte per year — PS010
   section 3).
4. The pool payout address `A_i` for the epoch (PS010 section 4).
5. Optionally, the `PSP1` registry announcement TXID for the epoch it
   approves.

The first three properties validate the approval; the last two are the
subject of the approval (the registry datum).

### 4.1 Update Transaction example

- `OP_RETURN { "cid": "bafybei...redacted...", "ts": 1770000000000 }`

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

A PS010 node (or any interested party) validates an approved registry datum as
follows:

1. Request the transaction history for the Reference Address, sorted
   descending by block height. Find the most recent transaction containing
   `APPROVAL` in its OP_RETURN — the latest Approval Transaction — and note
   the address which generated it (the multisig address).
2. Retrieve the Update Transaction from the TXID in the Approval
   Transaction's OP_RETURN, and the CID from the Update Transaction's
   OP_RETURN.
3. Fetch and parse the JSON document for that CID from an IPFS gateway.
4. Assert the JSON's `satsPerMb` and pool address `A_i` equal the registry
   epoch (or proposed epoch) being validated — PS010 section 4.2 / failure
   case F9.
5. Assert the JSON's public keys are exactly the registry's published council
   keys (or an authoritative council-key update), and recompute the P2SH
   multisig address from them using the published threshold.
6. Assert the recomputed multisig address equals the address that generated
   the Approval Transaction, and that the Approval Transaction's first input
   spends from that multisig address (i.e. it was actually signed by the n-of-m
   council keys).

No SLP token checks and no NFT checks are performed at any step. Validation is
pure BCH: on-chain public keys → multisig address → approval spends from it.

## 7. Differences from PS009 v1

| Aspect | v1 (1.0.2) | v2 (this revision) |
|---|---|---|
| Membership proof | holders of SLP NFTs of a PSF Group Token | node-operators' plain BCH public keys |
| Token dependency | PSF token / SLP ecosystem | none |
| Approving authority | PSF Minting Council | new registry council of node-operators |
| Approved datum | PSFFPP write-price | PS010 `satsPerMb` rate + pool address |
| Validation inputs | group token ID + NFT holdings | recomputed multisig address only |
| Use case | PSFFPP consensus-sync of write price | PS010 file-pinning registry epochs (§4.2) |

v1 remains valid for the systems already using it; this revision does not
withdraw it. It only defines the mechanism PS010 references for its long-term
registry governance. PS010 section 8.2's removal of *legacy PS009 token
write-price lookups* refers to PS009 v1's data feed, not to this approval
workflow.

## 8. Status

Draft — added alongside PS010 v2.0.0 (PR #11) for review and is **not yet
normative**. The library implementation of v1
([psf-multisig-approval](https://www.npmjs.com/package/psf-multisig-approval))
is expected to be revised in lockstep if this draft is adopted.
