

### Alice Pays Bob, in lightning

1. Bob creates secret P.
2. Bob sends H(P) to Alice.
3. Alice creates HTLC based on H(P) and updates the state.
   1. Alice sends update_add_htlc to Bob, this contains H(P)
   2. Bob sends commitment_signed, signing AUE2.
   3. Alice sends revoke_and_ack, revoking AUE1.
   4. Alice sends commitment_signed, signing BUE2.
   5. Bob sends revoke_and_ack, revoking BUE1.
4. Bob releases P, and triggers a state update 



