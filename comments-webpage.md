OpenARK has two types of users

1. End users, that mostly spend there capital and receive capital as part of a round.
2. ARK Services that provide different services, they co-sign for the ark, such as:
    1. Liquidity provider.
    2. Lightning GateWay.
    3. On/Off Board Swap service.

We should explain this. (It's not only the XLP).


The picture in "VTXOs, rounds, and trustless exits" Should be more like this. 



OpenArk vs other Ark implementations
Forfeit model	Funds are offboarded — no penalty seizure	ASP **can** take user funds as penalty

Not true, remove
Key model	Owner + agent key separation, per-round rekeying	Single key, no protocol-level delegation

Ours is also planned
Asset support	RGB integration (atomic with BTC in same round)	Arkade Assets framework (USDT planned)


This requires the sender + the ASP + a majority of XLP co-verifiers to all collude
This requires the sender + the ASP + **all** of co-verifiers to all collude

In-round payments = payments that happen inside a round.
Out-round payments = payments that happen outside a round.
You have mixed it up.


What happens if I'm offline during a round transition?

This is wrong.
Your funds stay in the old round and are recovered via the recycle transaction. You can sign the cooperative recycle when you come back online, or the ASP can broadcast the unilateral recycle after the recycle block height. Your capital is never lost. Alternatively, run a cloud agent to handle transitions automatically.

If you are offline, your agent transfers your funds safely to the next round, as per request. Your agent is also responsible for guarding against partial unilateral exits. 

We don't use the public nostr relays, every ASP uses their own relay.