# Understanding the Foundations of Bitcoin

First two weeks in the Open Source Fellowship with Btrust Builders focused on the history and technical foundations of Bitcoin. We looked at the research and ideas that existed before Bitcoin and how Satoshi Nakamoto brought many of them together into a functioning peer-to-peer electronic cash system.

## The Research Behind Bitcoin

One of the things that stood out in the first module was how much of Bitcoin's foundation came from research that was not originally intended for Bitcoin.

As explored in [Bitcoin's Academic Pedigree](https://queue.acm.org/detail.cfm?id=3136559), researchers had already been working on areas such as digital certificates, email signing, timestamping, proof of work, Byzantine fault tolerance and the use of public keys as identities.

These were separate areas of research solving different problems. Satoshi combined several of these ideas into a single system and used them to address the problem of creating a decentralised digital currency without relying on a central authority. The original [Bitcoin Whitepaper](https://chaincode.com/bitcoin.pdf) describes this system in full.

This made the history of Bitcoin much more interesting because it is not a collection of completely new technologies. It is the combination of existing cryptographic and distributed-systems ideas into a system with a specific purpose.

## A Brief History of How Bitcoin Got Here

Understanding the technical foundations also meant understanding how Bitcoin's development unfolded over time. [The Incomplete History of Bitcoin Development](https://b10c.me/blog/004-the-incomplete-history-of-bitcoin-development/) traces key events from the genesis block through major protocol upgrades, showing how the network has handled bugs, forks and community disagreements along the way.


## Thinking About a 51% Attack

We also discussed the incentives behind a **51% attack**.

If one entity controls a majority of the network's mining power, it can potentially reorganise parts of the chain and censor or reject transactions. However, having majority mining power does not give the attacker the ability to arbitrarily change Bitcoin's consensus rules.

This raised an important question: what does an attacker actually gain from controlling 51% of the mining power? The distinction between influencing the chain and changing protocol rules is a core theme in [Security Models with John Newbery](https://btctranscripts.com/chaincode-residency/2019-06-17-john-newbery-security-models).

## Simplified Payment Verification

Another concept covered was **Simplified Payment Verification (SPV)**.

An SPV client does not need to store the entire blockchain. Instead, it can keep the block headers of the longest proof-of-work chain and use them to verify that transactions are included in the blockchain, as described in section 8 of the [Bitcoin Whitepaper](https://chaincode.com/bitcoin.pdf).

This is an important part of understanding how users can interact with Bitcoin without every participant needing to maintain a complete copy of all blockchain data.

## Bitcoin's Circular Bootstrapping Problem

We also discussed what can be described as a circular dependency within Bitcoin's design.

A secure ledger depends on two distinct roles: miners who expend computational work to order transactions and extend the chain and full nodes that independently validate every block and transaction against Bitcoin's consensus rules. Miners need an incentive to perform this work. That incentive comes through block rewards and transaction fees. For the rewards to have value, however, the underlying currency needs to be valuable and trusted.

This creates a loop where the security of the ledger supports the value of the currency while the value of the currency provides an incentive to secure the ledger.

[Bitcoin's Academic Pedigree](https://queue.acm.org/detail.cfm?id=3136559) describes this as Nakamoto's true innovation — not any individual component, but the intricate way these components depend on and reinforce each other.

## Pruned Nodes and Chain Reorganisations

Pruned nodes were another interesting part of the module.

A pruned node removes older blockchain data after it is no longer needed for its normal operation. This raised an interesting question around chain reorganisations: if a reorganisation requires access to blockchain data that a pruned node has already deleted, how far can that node participate in the reorganisation?

The discussion highlighted an important tradeoff between reducing storage requirements and retaining historical blockchain data. The security implications of different node types are explored in depth in [Security Models with John Newbery](https://btctranscripts.com/chaincode-residency/2019-06-17-john-newbery-security-models).

We also looked at **Bloom filters**, introduced through BIP 37. Bloom filters allow lightweight clients to request relevant transactions without directly revealing all of the addresses they are interested in.

## Soft Forks, Hard Forks and SegWit

The second module focused on Segregated Witness, or **SegWit**, as well as the difference between soft forks and hard forks.

A hard fork changes the consensus rules in a way that can make previously valid blocks or transactions invalid under the new rules. This can result in nodes following different chains.

A soft fork introduces new consensus rules while remaining compatible with older nodes under certain conditions.

SegWit is an example of a soft fork. A clear introduction to how it works is available in [What is SegWit?](https://bitcoinmagazine.com/guides/what-is-segwit), with a deeper technical treatment in [SegWit in Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch06_transactions.adoc#segregated-witness).

## Transaction Malleability

One of the main problems SegWit addressed was **transaction malleability**.

Before SegWit, certain parts of a transaction could be modified by a third party without changing the actual effect of the transaction. However, these changes could alter the transaction ID. This created problems for systems that relied on the transaction ID remaining unchanged.

SegWit addressed this by separating the witness data from the transaction data used to calculate the transaction ID. As a result, changes to the witness no longer change the transaction ID.

SegWit also changed how transaction data is accounted for within a block. The full list of improvements this enabled is documented in [SegWit Benefits](https://bitcoincore.org/en/2016/01/26/segwit-benefits/), and the scalability implications are covered in [SegWit's Impact on Scalability](https://btctranscripts.com/scalingbitcoin/hong-kong-2015/segregated-witness-and-its-impact-on-scalability/).

## Non-SegWit Nodes

One question that came up during our home group discussion was whether a non-SegWit node can still be considered a full node, a question listed directly in the [SegWit discussion questions](https://chaincode.gitbook.io/seminars/bitcoin-protocol-development/segwit) on the Chaincode seminar page.

My understanding is that a non-SegWit node can see a SegWit transaction but does not understand or validate the SegWit-specific rules. From its perspective, the output can appear to be spendable by anyone.

This is an interesting example of how soft-fork upgrades maintain compatibility while newer nodes enforce rules that older nodes do not recognise.

## How Blocks Travel Across the Network

Once a miner finds a valid block, it needs to reach every other node on the network as quickly as possible. Every second of delay is a second other miners spend hashing the wrong problem and becomes wasted work that benefits no one.

The naive approach of sending the full block to every peer is expensive. Most transactions in a new block are already known to the network because they were broadcast individually when users first submitted them. Compact Blocks, described in [BIP 152](https://github.com/bitcoin/bips/blob/master/bip-0152.mediawiki), addresses this by sending only a compressed representation of the block: the header, short transaction identifiers and any transactions the sender predicts the peer does not already have. The receiving node reconstructs the full block from its own mempool and requests only what is missing. A roughly 9KB sketch can reconstruct a full 1MB block in the best case — a significant reduction in the bandwidth needed to keep the network in sync.

## Eclipse Attacks and the Limits of Peer Connections

A less obvious threat to a Bitcoin node is an eclipse attack. Rather than attacking the protocol directly, an attacker surrounds a victim node by controlling all of its peer connections. The victim only sees what the attacker wants it to see. Transactions can be censored in either direction, and the victim has no reliable way to detect that anything is wrong.

Bitcoin defends against this by maintaining diverse outbound connections across different IP ranges, valuing long-lasting peer relationships and allowing nodes to run on multiple network interfaces simultaneously. Each of these raises the cost of a successful eclipse. The tension, as discussed in [Amiti Uttarwar's talk on attacking Bitcoin Core](https://btctranscripts.com/la-bitdevs/2020-04-16-amiti-uttarwar-attacking-bitcoin-core), is that some defences against eclipse attacks work against privacy. Stable, long-lasting connections are harder to eclipse but easier for an observer to use to trace which transactions originated from your node. Bitcoin continues to navigate this tradeoff at the protocol level.

## Conclusion

The first two weeks' modules provided a much clearer picture of the technical foundations behind Bitcoin.

From the earlier research that influenced its design to proof of work, SPV, mining incentives, pruned nodes, soft forks, transaction malleability, SegWit, block propagation and eclipse attacks each concept connects to a particular problem in building and maintaining a decentralised monetary network.

What I found most interesting is how these different components interact. Understanding Bitcoin is not just about understanding individual concepts. It is also about understanding why each component exists and what problem it is solving.

---

## Further Reading

These are the primary resources from the [Chaincode Bitcoin Protocol Development Seminar](https://chaincode.gitbook.io/seminars/bitcoin-protocol-development/welcome-to-the-bitcoin-protocol) that informed this article.

**Bitcoin Foundations**
- [Bitcoin Whitepaper](https://chaincode.com/bitcoin.pdf) — Satoshi Nakamoto
- [Bitcoin's Academic Pedigree](https://queue.acm.org/detail.cfm?id=3136559) — Narayanan & Clark
- [The Incomplete History of Bitcoin Development](https://b10c.me/blog/004-the-incomplete-history-of-bitcoin-development/) — b10c
- [Security Models with John Newbery](https://btctranscripts.com/chaincode-residency/2019-06-17-john-newbery-security-models) — Chaincode Residency transcript

**SegWit**
- [What is SegWit?](https://bitcoinmagazine.com/guides/what-is-segwit) — Bitcoin Magazine
- [SegWit in Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch06_transactions.adoc#segregated-witness) — Andreas Antonopoulos
- [SegWit Benefits](https://bitcoincore.org/en/2016/01/26/segwit-benefits/) — Bitcoin Core
- [SegWit's Impact on Scalability](https://btctranscripts.com/scalingbitcoin/hong-kong-2015/segregated-witness-and-its-impact-on-scalability/) — Scaling Bitcoin transcript

**Network and Peer-to-Peer**
- [Compact Blocks FAQ](https://bitcoincore.org/en/2016/06/07/compact-blocks-faq/)
- [Amiti Uttarwar's talk on attacking Bitcoin Core](https://btctranscripts.com/la-bitdevs/2020-04-16-amiti-uttarwar-attacking-bitcoin-core) — Chaincode Residency transcript