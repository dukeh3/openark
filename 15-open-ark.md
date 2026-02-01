# OpenARK

### Design properties

1. HTLC-compatible model (must be part of a Lightning chain)
2. Support for RGB in ARK, define how the RGB model works in these channels.
3. Use nostr as a transport protocol.

### Expected outcome

1. A BOLT document explaining how Lightning can be extended to a VTXO-based multi-channel with a trusted party as a resolver (ARK-channel).
2. A NIP document that proposes a nostr-based transport model for the messages.

## Roles

### Ark Service Provider
    Operates the nostr relay and runs the ARK.

### Co-Verifier
    Has one of the keys to S, as well as their own User key. If a User is a Co-Verifier, that can be hidden. 

### User
    Has only their own User key.

## GuardTowers

ARK requires participants to sign during round closures. OpenARK solves the offline/inactive user problem through **GuardTowers**.

### Components

| Component | Function |
|-----------|----------|
| GuardTower Agent | Cloud-based agent that remains online on behalf of users |
| Activation Window | Only active during round closure bureaucracy |
| Handoff | Control returns to user wallet after round completion |

### How It Works

1. User delegates signing authority to their GuardTower for round closure operations
2. GuardTower remains online during round transitions
3. When a round closes, GuardTower signs on behalf of the user
4. After the round completes, control returns to the user's wallet
5. GuardTower only has authority during the activation window, minimizing trust requirements

This architecture is particularly suited for always-on cloud infrastructure, enabling users to participate in rounds even when their primary wallet is offline.

## Trustless Liquidity Provisioning

A key innovation in OpenARK is enabling external liquidity providers (LPs) to fund ASP rounds without custody risk to either party.

### The Problem

When an ASP closes a round, it needs sufficient liquidity to execute the VTXO swap. Small or new ASPs may lack the capital to service user demand.

### The Solution: Cryptographic LP Integration

External LPs can participate in round closures through multisig integration:

1. ASP needs liquidity to close round
2. LP offers to provide required capital
3. ASP includes LP's key in multisig structure (LP key becomes part of ASP key for this round)
4. LP independently verifies no double-spend conditions
5. Pre-signed contract ensures revocation TX pays LP's share to LP's key
6. Round closes — trustless for both parties

### Security Properties

| Property | Mechanism |
|----------|-----------|
| LP fund safety | LP key required in multisig; revocation pays to LP key |
| ASP non-custodial proof | LP can cryptographically verify ASP cannot rug users |
| Double-spend prevention | LP validates state before providing liquidity |
| Atomicity | Swap is all-or-nothing via pre-signed contracts |

The LP effectively becomes a temporary participant in the round's key structure, not a lender with counterparty risk. This enables ASPs to scale without proportional capital requirements while maintaining trustless guarantees.

## Examples

### Example 1.0 - Alice, Bob, Carol, Dave and Eve create a VTXO tree

In this example we let five Users Alice, Bob, Carol, Dave and Eve create a VTXO tree with Steve acting as the Ark Service Provider.

1. Steve issues a **new_round_initiate**
   1. Informs everybody that a new round is starting.
2. Alice, Bob, Carol, Dave and Eve issue a **new_round_join**
   1. Here the Users declare what to onboard, what to offboard and what to transition.
3. Steve issues a **new_round_vtxo_tree_proposal**
   1. Here Steve sends out the VTXO tree for signing.
4. Alice, Bob, Carol, Dave and Eve issue a **new_round_vtxo_tree_accept**
   1. Here the Users return the signed nodes.
5. Steve issues a **new_round_prepare_start**
   1. Here Steve sends out the new root for signature, binding the tree to the root.
6. Alice, Bob, Carol, Dave and Eve issue a **new_round_start_prepared**.
   1. Here all Users accept the tree.
7. Steve issues a **new_round_start**.
   1. The root is deposited and the round starts. This is the official start of the round.  

### Example 1.1 – Alice pays Bob

1. Bob creates secret P.
2. Bob sends H(P) to Alice.
3. Alice creates an HTLC based on H(P) and proposes a state update.
    1. Creates vtxo_spend_request, tagged as an HTLC_Offer.
    2. Steve signs vtxo_spend_complete.
4. Bob releases P.
    1. Creates vtxo_spend_request, tagged as a HTLC_Success.
    2. Steve signs vtxo_spend_complete.

### Example 1.2 – Round ends, Bob offboards 0.5 BTC, Dave is offline and misses the round, Eve onboards 0.4 BTC.

1. The block designated as the closing block at the start of the round is mined. 
2. Steve issues a **new_round_initiate** message signaling a new round.
3. Steve issues **round_closed**. He tags the **new_round_initiate** message indicating the round to be transferred to.
4. The Users decide how to proceed with the new round, and they each issue a **new_round_join** message.
   1. Alice indicates that she wants to transfer all of her capital. 
   2. Bob indicates that he wants to offboard 0.5 BTC. 
   3. Carol indicates that she wants to transfer all of her capital. 
   4. Dave is offline and does not send a message, hence missing the round. His capital is left in the closed round and will be recycled. 
   5. Eve indicates that she wants to transfer all her current capital and also to onboard 0.4 BTC.
5. Steve collects the responses and, after the timeout is reached, issues a **new_round_vtxo_tree_proposal**. This includes: 
   1. A new round transaction containing:
      1. A new VTXO root output, locked with A+B+C+E+S | S + T=2000.
      2. An offboarding output for Bob
      3. A forfeit root output.
      4. A funding input for Eve.
      5. A funding input for Steve.
   2. Forfeit control transaction.
   3. Forfeit transactions for the capital that is being transferred.
   4. A recycle transaction for the round containing:
      1. An output with Dave's capital based on his recycle address.
      2. An output to Steve with the rest of the capital.
6. The Users, Alice, Bob, Carol and Eve verify that the tree matches the request, and then each issues a **new_round_vtxo_tree_accept**. It contains:
   1. Signatures for the new VTXO tree.
7. Steve verifies that everything is there and then issues a new_round_funding_request.
8. The Users, Alice, Bob, Carol and Eve verify that everything is OK, and then each issues a **new_round_funding_accept**. It contains:
   1. Signatures for the forfeit transactions.
   2. Signatures for the funding transactions (only Eve has one).
   3. Signatures for the recycle transaction.
9. Steve verifies that the funding has been received, commits the transaction, and then issues a **new_round_start**.
10. Dave comes online, understands that he missed the round, and issues a **recycle_accept** containing his signature for the recycle transaction.
11. Steve submits the recycle transaction.

### Funding a round

There are two ways for Users to fund a round, either by signing over capital from a closed round to the Ark Service Provider using a forfeit transaction or by onboarding new capital.

#### Forfeit transactions

A forfeit transaction is a way for Users to swap capital from a recently closed round into a new round in a trustless way,
where depositing the new round root transaction is the atomic action triggering the swap. When Steve proposes a new round, 
he adds a forfeit root output to the round root transaction. This root is connected to the forfeit trunk and from there 
to forfeit leaves. When a forfeit transaction is created, a forfeit leaf is added as one of its inputs, along with the capital 
being forfeited, assuring the User that the only way this forfeit transaction can be activated is if the round root is deposited. 
If the root is not deposited, then the forfeit transaction is invalid, and the User can still do a unilateral exit.

#### Recycle transactions

A recycle transaction is a collaborative way to recycle capital from closed rounds that transfer capital to a new round. 
When the new VTXO tree is created, an optional set of recycle transactions can be added to the **new_round_vtxo_tree_proposal**. 
These transactions provide a collaborative way to recycle capital from closed rounds. The transactions are designed as follows:
1. All User capital that is not being transferred to the new round is sent to a recycle address provided by the respective User at round start.
2. The rest is distributed to the Ark Service Provider.

In most rounds, where all Users are online at the end of the round, all capital is moved to the new round, and the only capital left 
belongs to the Ark Service Provider.

## RGB

Here are examples that add support for RGB.

Block 100 is mined. This starts the end of the round. 

### Example 2.1 - Creating a VTXO tree

![Architecture overview](./vtxo-tree.svg)

In this example we let five Users Alice, Bob, Carol, Dave and Eve create a VTXO tree with Steve acting as the Ark Service Provider. The tree has two values, the first is the bitcoin value of the transaction, the second is an RGB asset value, Example Coin (EC). Now RGB has a lot more flexibility than regular UTXOs but for the sake of this example we assume the following, the RGB state transitions follow the BTC as marked in the diagram.

### Example 2.2 - Alice makes a cross atomic swap with Bob.

This example uses the VTXO tree in Example 1. The goal of this example is to create an HTLC transaction between Bob and Alice, swapping 0.2 BTC for 0.2 EC. 

![Architecture overview](./a-swap-with-b.svg)

1. Alice creates a secret P
2. Alice creates an HTLC VTXO leaf (HTLC 1) where one of the outputs is locked with H(P). The output is created in such a way that Bob can claim the VTXO either on-chain using a unilateral exit or in-ARK if and only if he reveals P to S.
3. Alice sends the transaction to Bob (This should be a nostr-message that is public)
4. Bob verifies the transaction, and then issues a counter transaction using the same H(P), HTLC 2. 
5. Bob sends the transaction to Alice. (This should be a nostr-message that is public)
6. Alice requests the Ark Service Provider to cosign the transition from HTLC 1 to VTXO A2, revealing P to Bob.
7. The Ark Service Provider grants Alice's request, creating VTXO A2.
8. Bob requests the Ark Service Provider to cosign the transaction from HTLC 2 to VTXO B2.
9. The Ark Service Provider grants Bob's request, creating VTXO B2.

### Example 2.3 - Alice fails a cross atomic swap with Bob, timeout triggered.

1. Alice creates a secret P
2. Alice creates an HTLC VTXO leaf (HTLC 1) where one of the outputs is locked with H(P). The output is created in such a way that Bob can claim the VTXO either on-chain using a unilateral exit or in-ARK if and only if he reveals P to S.
3. Alice sends the transaction to Bob (This should be a nostr-message that is public)
4. Bob verifies the transaction, and then issues a counter transaction using the same H(P), HTLC 2.
5. Bob sends the transaction to Alice. (This should be a nostr-message that is public)
6. Alice fails to request cosignature from the Ark Service Provider in time. Timeout is triggered.
7. Bob requests the Ark Service Provider to cosign the transaction from HTLC 2 to VTXO B2.
8. The Ark Service Provider grants Bob's request, creating VTXO B2.
9. Alice requests the Ark Service Provider to cosign the transition from HTLC 1 to VTXO A2.
10. The Ark Service Provider grants Alice's request, creating VTXO A2.
