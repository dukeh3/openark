# OpenARK

### Design properties

1. HTLC compatible model (must be part of a lightning chain)
2. Support for RGB in ARK, define how the RGB model works in these channels.
3. Use nostr as transport protocol.

### Expected outcome

1. A Bolt document explaining how lightning can be extended into a VTXO based multi-channel with a trusted party as a resolver (ARK-channel).
2. A NIP document that proposes a nostr based transport model for the messages.

## Roles

### Ark Service Provider
    Operates the nostr relay and runs the ARK.

### Co Verifier
    Has one of the keys to S, as well as their own user key. if a user is a co-verifier can be hidden. 

### User
    Has only his own user key. 

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

### Example 1.1 – Alice pays Bob

1. Bob creates secret P.
2. Bob sends H(P) to Alice.
3. Alice creates HTLC based on H(P) and proposes a state updates to the state.
    1. Creates vtxo_spend_request, tagged as a HTLC_Offer.
    2. Steve signs vtxo_spend_complete.
4. Bob releases P.
    1. Creates vtxo_spend_request, tagged as a HTLC_Success.
    2. Steve signs vtxo_spend_complete.

### Example 1.3 – Round ends, Bob offboards 0.5 BTC, Dave is offline and misses the round, Eve onboards 0.4 BTC.

1. The block named as the closing block at the start of the round is mined. 
2. Steve issues a new_round_initiate message signaling a new round.
3. Steve issues round_closed_message, he tags the new_round_initiate message indicating the round to be transferred to.
4. The users decide how to proceed with the new round, they each issue a new_round_join message.
   1. Alice indicates that she wants to transfer all of her capital. 
   2. Bob indicates that he wants to offboard 0.5 BTC. 
   3. Carol indicates that she wants to transfer all of her capital. 
   4. Dave is offline and does not send a message, hence missing the round. His capital is left in the closed round and will be recycled. 
   5. Eve indicates that she wants to transfer all her current capital and also to onboard 0.4 BTC.
5. Steve collects the responses, and after the timeout is reached issues a new_round_vtxo_tree_proposal. This contains: 
   1. A new round transaction containing:
      1. A new VTXO root output, locked with A+B+C+E+S | S + T=2000.
      2. An offboarding output for Bob
      3. A forfeit output root.
      4. A funding input for Eve.
      5. A funding input for Steve.
   2. Forfeit control transaction.
   3. Forfeit transactions for the capital that is being transferred.
   4. A recycle transaction for the round containing:
      1. An output with Dave's capital based on his recycle address.
      2. An output to Steve with the rest of the capital.
6. The users, Alice, Bob, Carol and Eve verify that the tree matches the request, and then each issue a new_vtxo_tree_accept, it contains:
   1. Signatures for the new VTXO tree.
7. Steve verifies that everything is here and then issues a new_round_funding_request.
8. The users, Alice, Bob, Carol and Eve verify that everything is ok, and then each issue a new_round_funding_accept, it contains:
   1. Signatures for the forfeit transactions.
   2. Signatures for the funding transactions (only Eve has one).
   3. Signatures for the recycle transaction.
9. Steve verifies that the funding has been received, commits the transaction and then issues a new_round_start.
10. Dave comes online, understands that he missed the round issues a recycle_accept containing his signature for the recycle transaction.
11. Steve submits the recycle transaction.

### Funding a round

There are two ways for users to fund a round, either by signing over capital from a closed round to the ark service provider using a forfeit transaction or by onboarding new capital.

#### Forfeit transactions

A forfeit transaction is a way for users to swap capital from a recently closed round into a new round in a trustless way,
where depositing the new round root transaction is the atomic action triggering the swap. When Steve proposes a new round, 
he will add a forfeit root output to the round root transaction. This root is connected to the forfeit trunk and from there 
to forfeit leaves. When a forfeit transaction is created, a forfeit leaf is added as one of its inputs, along with the capital 
being forfeited, assuring the user that the only way this forfeit transaction can be activated is if the round root is deposited. 
If the root is not deposited, then the forfeit transaction will be invalid, and the user can still do a unilateral exit.

#### Recycle transactions

A recycle transaction is a collaborative way to recycle the capital in closed rounds that transfer capital to a new round. 
When the new VTXO tree is created, an optional set of recycle transactions can be added to the new_round_vtxo_tree_proposal. 
These transactions provide a collaborative way to recycle the capital in the closed rounds. The transactions are designed as follows:
1. All user capital that is not being transferred to the new round is sent to a recycle address provided by the respective user at round start.
2. The rest is distributed to the ark service provider.

In most rounds, where all the users are online at the end of the round, all the capital is moved to the new round, and hence the 
only capital left will belong to the ark service provider.

## RGB

Here are examples that add support for RGB.

Block 100 is mined, this starts the end of the round. 

### Example 2.1 - Creating a VTXO Tree

![Architecture overview](./vtxo-tree.svg)

In this example we let five users Alice, Bob, Carol, Dave and Eve create a VTXO Tree with Steve acting as the Ark Service provider. The tree has two values the first is the bitcoin value of the transaction, the second is an RGB asset value Example Coin (EC). Now RGB has a lot more flexibility that regular UTXO:s but for the sake of this example we assume the following, the RGB state transition follow the BTC as marked in the diagram.

### Example 2.2 - Alice makes a cross atomic swap with Bob.

This example uses the VTXO tree in example 1. The goal of this example is to create a HTLC transaction between Bob and Alice, swapping 0.2 BTC for 0.2 EC. 

![Architecture overview](./a-swap-with-b.svg)

1. Alice creates a secret P
2. Alice creates a HTLC VTXO leaf (HTLC 1) where one of the outs is locked with H(P), the output is created in such a way that Bob can claim the VTXO either on-chain using a unilateral exit or in-ark if and only if he reveals P to S.
3. Alice sends the transaction to Bob (This should be a nostr-message that is public)
4. Bob verifies the transaction, and then issues a counter transaction using the same H(P), HTLC 2. 
5. Bob sends the transaction to Alice. (This should be a nostr-message that is public)
6. Alice requests to Service-provider to cosign the transition from HTLC 1 to VTXO A2, revealing P to Bob.
7. Service-provider grants Alice request, creating VTXO A2.
8. Bob requests to Service-provider to cosign the transaction from HTLC 2 to VTXO B2.
9. Service-provider grants Bob's request, creating VTXO B2.

### Example 2.3 - Alice fails a cross atomic swap with Bob, timeout triggered.

1. Alice creates a secret P
2. Alice creates a HTLC VTXO leaf (HTLC 1) where one of the outs is locked with H(P), the output is created in such a way that Bob can claim the VTXO either on-chain using a unilateral exit or in-ark if and only if he reveals P to S.
3. Alice sends the transaction to Bob (This should be a nostr-message that is public)
4. Bob verifies the transaction, and then issues a counter transaction using the same H(P), HTLC 2.
5. Bob sends the transaction to Alice. (This should be a nostr-message that is public)
6. Alice fails to request cosignature from Service-provider in time. Timeout is triggered.
8. Bob requests to Service-provider to cosign the transaction from HTLC 2 to VTXO B2.
9. Service-provider grants Bob's request, creating VTXO B2.
10. Alice requests to Service-provider to cosign the transition from HTLC 1 to VTXO A2.
11. Service-provider grants Alice request, creating VTXO A2.
