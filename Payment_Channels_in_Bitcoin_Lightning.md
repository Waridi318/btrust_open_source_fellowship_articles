Structure

# Payment Channels in Bitcoin Lightning: How Two People Transact Without Touching the Blockchain

## The problem: Why Bitcoin's Base Layer Cannot Handle Everyday Payments

The first reason is limited block space on-chain. Bitcoin blocks have a maximum size of 4 megabytes in weight units. Miners prioritize transactions with the highest fees, which means during periods of high demand, lower-fee transactions either wait long periods or get priced out entirely. This severely limits the number of transactions that can be processed on-chain at any given time.

The second reason is latency. On average, a new block is mined every ten minutes. This means any transaction you broadcast must wait roughly ten minutes before it is included in a block, and for a meaningful amount you would typically want several confirmations on top of that. Nobody waits an hour to buy a coffee.

Combined, these two constraints give Bitcoin a throughput of roughly seven transactions per second globally. For comparison, Visa processes tens of thousands of transactions per second. The gap is enormous.

With these limitations, it becomes clear that Bitcoin's base layer alone cannot serve as an everyday payment network. This is the problem Lightning was built to solve.

Section 2: The core idea
Off-chain state, on-chain settlement. 
The blockchain as a court not a cashier.

Section 3: The funding transaction
What a channel actually is. 
Why Alice cannot just send to that script and walk away.

Section 4: The five-messages
Walking through open_channel, accept_channel, funding_created, funding_signed, channel_ready. 
Why the ordering matters. 
Why the commitment transaction must exist before the funding transaction is broadcast.

Section 5: Commitment transactions
What they look like. 
The to_remote and to_local outputs. 
Why they are asymmetric. 
What the timelock on your own output actually does.

Section 6: Updating state
Alice pays Bob 1 BTC. 
New commitment transactions. 
The old state problem. Revocation via private key exchange. 
Why handing over your old key makes cheating suicidal.

Section 7: Closing the channel
Three ways it ends. 
Cooperative close, force close, breach close.

Section 8: The big picture
Two on-chain transactions. 
Unlimited payments in between. What Lightning is actually doing.