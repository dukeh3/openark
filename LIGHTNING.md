# Lightning vs OpenARK Payment Flows

This document compares the payment flow mechanics between standard Lightning Network channels and OpenARK VTXO-based channels. Understanding these differences is essential for implementing HTLC compatibility between the two systems.

## Overview

Both systems enable off-chain Bitcoin payments using hash time-locked contracts (HTLCs), but they differ in their state management and commitment mechanisms:

| Aspect | Lightning | OpenARK |
|--------|-----------|---------|
| State Model | Bilateral commitment transactions | VTXO tree with ASP co-signing |
| Revocation | Revoke-and-ack exchange | No revocation needed (append-only) |
| Intermediary | None (peer-to-peer) | Ark Service Provider |
| Message Transport | Lightning protocol (BOLT) | Nostr (NIP-150) |

## Lightning: Alice Pays Bob

In standard Lightning, payment requires a four-message commitment exchange to update channel state safely.

### Flow

1. Bob creates secret P.
2. Bob sends H(P) to Alice.
3. Alice creates HTLC based on H(P) and proposes a state update.
   1. Alice sends `update_add_htlc` to Bob, containing H(P).
4. Alice initiates a state transition:
   1. Alice sends `commitment_signed`, signing BUE2.
   2. Bob sends `revoke_and_ack`, revoking BUE1.
   3. Bob sends `commitment_signed`, signing AUE2.
   4. Alice sends `revoke_and_ack`, revoking AUE1.
5. Bob releases P and triggers a new proposal:
   1. Bob sends `update_fulfill_htlc`.
6. Bob initiates a new state update:
   1. Bob sends `commitment_signed`, signing AUE3.
   2. Alice sends `revoke_and_ack`, revoking AUE2.
   3. Alice sends `commitment_signed`, signing BUE3.
   4. Bob sends `revoke_and_ack`, revoking BUE2.

### Key Characteristics
- **Bilateral**: Only Alice and Bob are involved
- **Revocation Required**: Each party must revoke old states
- **Synchronous**: State updates require round-trip exchanges

## OpenARK: Alice Pays Bob

In OpenARK, payments are processed through VTXO spend requests co-signed by the Ark Service Provider.

### Flow

1. Bob creates secret P.
2. Bob sends H(P) to Alice.
3. Alice creates an HTLC based on H(P) and proposes a state update.
   1. Creates `vtxo_spend_request`, tagged as `HTLC_Offer`.
   2. Steve (ASP) signs `vtxo_spend_complete`.
4. Bob releases P.
   1. Creates `vtxo_spend_request`, tagged as `HTLC_Success`.
   2. Steve (ASP) signs `vtxo_spend_complete`.

### Key Characteristics
- **Trilateral**: Alice, Bob, and ASP are involved
- **No Revocation**: VTXO model is append-only
- **Simpler Flow**: Fewer message exchanges required
- **ASP Dependency**: Requires online Ark Service Provider

## Comparison Summary

### Message Count
| Action | Lightning | OpenARK |
|--------|-----------|---------|
| HTLC Setup | 5 messages | 2 messages |
| HTLC Fulfill | 5 messages | 2 messages |
| **Total** | 10 messages | 4 messages |

### Trade-offs

**Lightning Advantages:**
- Fully decentralized (no third party)
- Established ecosystem and tooling
- Proven security model

**OpenARK Advantages:**
- Simpler message flow
- No revocation complexity
- Better suited for multi-party scenarios
- RGB asset support

## HTLC Compatibility

For OpenARK to participate in Lightning payment routes, the HTLC semantics must be compatible. Both systems use:
- Hash locks: H(P) for conditional payments
- Time locks: Timeout conditions for failed payments
- Preimage reveal: P to claim funds

The key difference is in state commitment and revocation, which is abstracted at the HTLC level, allowing interoperability.

## See Also

- [Main Specification](./15-open-ark.md) - Full OpenARK protocol specification
- [Message Format](./nip-150.md) - Nostr message format for OpenARK transactions
