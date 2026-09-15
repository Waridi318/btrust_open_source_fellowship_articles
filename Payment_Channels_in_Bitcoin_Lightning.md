# Payment Channels in Bitcoin Lightning: How Two People Transact Without Touching the Blockchain

---

## Why Bitcoin's Base Layer Cannot Handle Everyday Payments

Bitcoin blocks are capped at roughly 1 MB of transaction data. Miners pick the highest-fee transactions first, so when demand is high, fees spike and lower-priority transactions wait. On top of that, a new block is mined only every ten minutes on average. Together, these constraints give Bitcoin a throughput of about seven transactions per second globally. Visa handles tens of thousands. The gap is the problem Lightning was built to solve.

---

## The Core Idea: Off-Chain State, On-Chain Settlement

Instead of putting every payment on-chain, Lightning locks funds in a single on-chain transaction, then lets two parties exchange payments freely off-chain. In the simplest case, the blockchain sees two channel transactions: one to open the channel and one to close it, regardless of how many payments happen in between.

The Bitcoin blockchain is not a cashier processing every purchase. It is a court you appeal to only when you need to settle up or resolve a dispute.

![Channel Lifecycle Flow](channel_lifecycle_flow-1.png)

---

## The Funding Transaction: What a Channel Actually Is

A channel is an unspent transaction output locked in a 2-of-2 multisig. Both signatures are required to spend it, so neither party can move the funds alone.

Alice cannot simply send funds to that multisig and call the channel open. If Bob disappears after she commits her money, she has no way to recover it without his signature.

The fix is ordering. Before Alice broadcasts the funding transaction, both parties first create and sign commitment transactions. These spend the funding output and pay each party their correct balance. Alice holds Bob's signature. Bob holds Alice's. Each has a valid exit they can broadcast unilaterally if the other disappears. Only then does Alice broadcast the funding transaction.

---

## Commitment Transactions: How Channel State Is Recorded

Each party holds their own version of the commitment transaction and the two versions are not identical. This asymmetry is what makes blame attribution possible: when a transaction appears on-chain, it is always clear whose version it is.

Each commitment transaction has two outputs:

- **`to_remote`** pays your counterparty immediately, no conditions.
- **`to_local`** pays you, but only after a `to_self_delay` block delay, or immediately by your counterparty if they hold the revocation key for that state.

![Commitment Transaction Structure](commitment_tx_structure-1.png)

Your own output is always the delayed one. That delay is the window your counterparty needs to punish you if you broadcast a revoked state. Initially they do not have your revocation key, so the punishment path is inactive. It activates only when you hand the key over during a state update.

---

## Updating State: Alice Pays Bob 1 BTC

Alice and Bob generate fresh key pairs, share public keys and build new commitment transactions reflecting updated balances: Alice 4 BTC, Bob 6 BTC. Then they each hand over the revocation key for the previous state. The old commitment transactions are now dead. Broadcasting either one hands the other party the revocation key they need to sweep the entire channel instantly.

![State Update and Revocation](state_update_revocation-1.png)

This repeats for every payment. The channel moves to a new commitment state, the corresponding per-commitment secrets are updated and the previous state is revoked. Once a state is revoked, broadcasting it allows the counterparty to use the revocation mechanism to claim the funds. The channel can continue updating without any on-chain activity.

---

## Closing a Channel

**Collaborative close:** Both parties online, agree on final balances, sign a clean transaction with no timelock. Funds available immediately.

**Force close:** One party offline. The other broadcasts their latest commitment transaction and waits out `to_self_delay` before spending their output.

**Breach close:** One party broadcasts a revoked state. The other uses the revocation key to sweep the entire channel balance. The cheater loses everything.

---

## The Big Picture

Two on-chain transactions. Unlimited payments in between. No miner fees per payment, no blocks to wait for, settlements in milliseconds.

Payment channels alone only connect two people directly. The real power of Lightning is routing payments across a network of channels to reach anyone. That is where HTLCs and the multi-hop layer come in, and that is a topic for another article.