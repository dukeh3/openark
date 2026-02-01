# OpenARK

OpenARK is a specification for extending Bitcoin's Lightning Network with VTXO-based (Virtual Transaction Output) multi-channel architecture. It enables trust-minimized off-chain transactions through an Ark Service Provider model while maintaining HTLC compatibility with existing Lightning infrastructure.

## Overview

OpenARK combines three key technologies:
- **ARK Protocol**: VTXO-based transaction model with collaborative rounds
- **Lightning Network**: HTLC-compatible payment channels
- **Nostr**: Decentralized transport protocol for messaging

## Design Philosophy

- **Wallet sovereignty**: Wallets can run their own ASP infrastructure rather than onboarding onto third-party ASPs
- **No vendor lock-in**: Decentralized architecture where wallets maintain control
- **Liquidity separation**: External liquidity providers can fund ASP rounds without controlling the ASP itself
- **Open messaging**: Nostr-based transport provides transparent, auditable communication

## Design Properties

1. HTLC-compatible model (must be part of a Lightning chain)
2. Support for RGB in ARK, defining how the RGB asset model works in these channels
3. Use Nostr as a transport protocol (see [NIP-150](./nip-150.md))

## Expected Outcomes

1. **BOLT Document**: Specification for extending Lightning to support VTXO-based multi-channel with a trusted party as resolver (ARK-channel)
2. **NIP Document**: Nostr-based transport model for OpenARK messages

## Documentation

| Document | Description |
|----------|-------------|
| [15-open-ark.md](./15-open-ark.md) | Main specification with roles, examples, and transaction types |
| [nip-150.md](./nip-150.md) | Nostr message format specification (kind 150) |
| [LIGHTNING.md](./LIGHTNING.md) | Comparison of Lightning vs OpenARK payment flows |

## Visual Resources

- [VTXO Tree Diagram](./vtxo-tree.svg) - Structure of a VTXO tree with multiple users
- [Atomic Swap Diagram](./a-swap-with-b.svg) - Cross-atomic swap between Alice and Bob

## Key Concepts

### Roles
- **Ark Service Provider (ASP)**: Operates the Nostr relay and runs the ARK
- **Co-Verifier**: Holds one key to the shared signature plus their own User key
- **User**: Holds only their own User key

### Transaction Types
- **Round Messages**: new_round_initiate, new_round_join, new_round_vtxo_tree_proposal, etc.
- **VTXO Messages**: vtxo_spend_request, vtxo_spend_confirmed, vtxo_spend_complete
- **HTLC Templates**: HTLC_Offer, HTLC_Success, HTLC_Timeout

## Differentiation

| Aspect | OpenARK | Other Implementations |
|--------|---------|----------------------|
| ASP operation | Wallet-operated | Centralized ASP recruits wallets |
| Fee capture | Wallet keeps majority | ASP captures fees |
| Liquidity model | Trustless external LPs | ASP must self-fund |
| Wallet dependency | None (run your own) | High (on ASP availability) |
| Messaging | Nostr (open, auditable) | Varies (JSON-RPC, gRPC) |

## Current Status

| Aspect | Status |
|--------|--------|
| Design | Complete |
| Testnet | Running |
| Mainnet | Not ready |

## Getting Started

1. Read the [main specification](./15-open-ark.md) to understand the protocol
2. Review the [message format specification](./nip-150.md) for Nostr integration
3. Study the examples in the main spec:
   - Example 1.0: Creating a VTXO tree
   - Example 1.1: Alice pays Bob
   - Example 1.2: Round transitions with onboarding/offboarding

## Related Projects

- [ARK Protocol](https://ark-protocol.org) - Original ARK specification
- [Lightning Network](https://lightning.network) - Bitcoin Layer 2 payment channels
- [Nostr Protocol](https://nostr.com) - Decentralized messaging protocol
- [RGB Protocol](https://rgb.tech) - Smart contracts on Bitcoin

## Contributing

This is a specification project. Contributions are welcome via pull requests for:
- Clarifications to existing specifications
- Additional examples
- Diagram improvements
- Protocol extensions
