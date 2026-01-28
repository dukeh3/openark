# OpenARK

### Design properties properties

1. HTLC compatible model (must be part of a lightning chain)
2. Support for RGB in ARK, define how the RGB model works in these channels.
3. Use nostr as transport protocoll.

### Expected outcome

1. A Bolt document explaining how lightning can be extended into a VTXO based multi-channel with a trusted party as a resolver (ARK-channle).
2. A NIP document that proposes a nostr based transport model for the messages.

## Roles

### Ark Service Provider
    Operates the nostrrelay and runs the ARK.

### Co Verifier
    Has one of the keys to S, aswell as there own user key. if a user is a coverifier can be hidden. 

### User
    Has only his own userkey. 

## Examples

### Example 1.0 - Alice, Bob, Carol, Dave and Eve create a VTXO tree

In this example we let five users Alice, Bob, Carol, Dave and Eve create a VTXO Tree with Steve acting as the Ark Service provider.

1. Steve issues a new_round_initiate
   1. Informs everybody that a new round is starting.
2. Alice, Bob, Carol, Dave and Eve issues a new_round_join
   1. Here the users declare what to onboard, what to offboard and what to transition.
3. Steve issues a new_round_vtxo_tree_proposal
   1. Here we send out the VTXO tree for signing.
4. Alice, Bob, Carol, Dave and Eve issues a new_vtxo_tree_accept
   1. Here we return the signed nodes.
5. Steve issues a new_round_start_confirmation_request
   1. Here we send out the new root for signature, binding the tree to the root.
6. Alice, Bob, Carol, Dave and Eve issues a new_round_start_confirmed.
   1. Here everybody accepts the tree.
7. Steve issues a new_round_start.
   1. The root is deposited and the round starts. This is the official start of the round.  

### Example 1.1 - Alice pays Bob

1. Bob creates secret P.
2. Bob sends H(P) to Alice.
3. Alice creates HTLC based on H(P) and proposes a state updates to the state.
    1. Creates vtxo_spend_request, tagged as a HTLC_Offer.
    2. Steve signs vtxo_spend_complete.
4. Bob releases P.
    1. Creates vtxo_spend_request, tagged as a HTLC_Success.
    2. Steve signs vtxo_spend_complete.

## RGB

Here are examples that add support for RGB.

### Example 2.1 - Creating a VTXO Tree

![Architecture overview](./vtxo-tree.svg)

In this example we let five users Alice, Bob, Carol, Dave and Eve create a VTXO Tree with Steve acting as the Ark Service provider. The tree has two values the first is the bitcoin value of the transaction, the second i an RGB asset value Example Coin (EC). Now RGB has a lot more flexibility that regualr UTXO:s but for the sake of this example we assume the following, the RGB state transition follow the BTC as marked in the diagram.

### Example 2.2 - Alice makes a cross atomic swap with Bob.

This example uses the VTXO tree in example 1. The goal of this example is to create a HTLC transaction between Bob and Alice, swaping 0.2 BTC for 0.2 EC. 

![Architecture overview](./a-swap-with-b.svg)

1. Alice creates a secret P
2. Alice creates a HTLC VTXO leaf (HTLC 1) where one of the outs is locked with H(P), the output is created in such a way that Bob can claim the VTXO either on-chain using a unilateral exit or in-ark if and only if he reveals P to S.
3. Alice sends the transaction to Bob (This shoud be a nostr-message that is public)
4. Bob verifies the transaction, and then issues a counter transaction using the same H(P), HTLC 2. 
5. Bob sends the transaction to Alice. (This should be a nostr-message that is public)
6. Alice requests to Service-provider to cosign the transition from HTLC 1 to VTXO A2, revealing P to Bob.
7. Service-provider grants Alice request, creating VTXO A2.
8. Bob requests to Service-provider to cosign the transaction from HTLC 2 to VTXO B2.
9. Service-provider grants Bobs request, creating VTXO B2.

### Example 2.3 - Alice fails a cross atomic swap with Bob, timeout triggered.

1. Alice creates a secret P
2. Alice creates a HTLC VTXO leaf (HTLC 1) where one of the outs is locked with H(P), the output is created in such a way that Bob can claim the VTXO either on-chain using a unilateral exit or in-ark if and only if he reveals P to S.
3. Alice sends the transaction to Bob (This shoud be a nostr-message that is public)
4. Bob verifies the transaction, and then issues a counter transaction using the same H(P), HTLC 2.
5. Bob sends the transaction to Alice. (This should be a nostr-message that is public)
6. Alice fails to request cosignature from Service-provider in time. Timeout is triggered.
8. Bob requests to Service-provider to cosign the transaction from HTLC 2 to VTXO B2.
9. Service-provider grants Bobs request, creating VTXO B2.
10. Alice requests to Service-provider to cosign the transition from HTLC 1 to VTXO A2.
11. Service-provider grants Alice request, creating VTXO A2.