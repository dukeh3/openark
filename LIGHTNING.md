

### Alice Pays Bob, in lightning

1. Bob creates secret P.
2. Bob sends H(P) to Alice.
3. Alice creates HTLC based on H(P) and proposes a state updates the state.
   1. Alice sends update_add_htlc to Bob, this contains H(P)
4. Alice is finished with here proposed updates and initiates a state transition 
   1. Alice sends commitment_signed, signing BUE2.
   2. Bob sends revoke_and_ack, revoking BUE1.
   3. Bob sends commitment_signed, signing AUE2.
   4. Alice sends revoke_and_ack, revoking AUE1.
5. Bob releases P, and triggers a new proposal. 
   1. Bob sends update_fulfill_htlc
6. Bob is finished with his proposals and triggers a new state update.
   1. Bob sends commitment_signed, signing AUE3.
   2. Alice sends revoke_and_ack, revoking AUE2.
   3. Alice sends commitment_signed, signing BUE3.
   4. Bob sends revoke_and_ack, revoking BUE2.

### Alice pays Bob, in OpenARK

1. Bob creates secret P.
2. Bob sends H(P) to Alice.
3. Alice creates HTLC based on H(P) and proposes a state updates the state.
   1. Creates vtxo_spend_request, tagged as a HTLC_Offer.
   2. Steve signs vtxo_spend_complete.
4. Bob releases P.
   1. Creates vtxo_spend_request, tagged as a HTLC_Success.
   2. Steve signs vtxo_spend_complete.

## OpenARK Templates

### HTLC_Offer

### HTLC_Success

### HTLC_Timeout

## Message format

### vtxo_spend_request

nostr_id: 6661
content: Base64 Encoded version of a PSBT, including signatures to all the incoming transactions as presented by the party.

Issues by the user that wish to create a new transaction.

    1. Tags all the outputs that are spent.
    2. Tags all the transaction that are spent.
    3. Tags all the signers that are requested to contribute.
    4. Tags all the potential beneficiaries. 
    5. Tags the template
 
   
### vtxo_spend_confirmed

nostr_id: 6662
content: Base64 Encoded version of a PSBT, including additional signature that this party provides.

Issues by co-signers that needs to be part of the transaction.

### vtxo_spend_complete

nostr_id: 6663
content: Base64 Encoded version of the transaction

Issued by Steve (S), contains the new transaction.

    1. Tags the txid of the new transaction.
    2. Tags to vtxo_spend_request.

### vtxo_spend_recall_request
nostr_id: 6664

### vtxo_spend_recalled
nostr_id: 6665



1. Issued by Steve (S), contains the new transaction.
    1. Tags the txid of the new transaction.
    2. Tags to vtxo_spend_request.


### Code for Steave

Only reactive, when you are tagged in a vtxo_spend_request you subscribe to the thread. You check if the current thread has enough information to complete the transaction. IE, you go over all of the inputs that are beeing spent and check:
1. Verify that the input is open for spending.
2. Check that all the signatures and other attributes are there.

If so, then you complete it, this means mark all the inputs as spent. Then send out the complete message and then unsubscribe from the thread.  





