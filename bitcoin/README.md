# Summary
Bitcoin is an electronic currency that is "created" every ~10 minutes, and owned by the "miner". Transfers of bitcoin between owners can occur.

Bitcoin tries to solve the problem of equivocation, or "double-spending" of money. The general approach is to make a transaction log that is public (transactions are signed by public keys), append-only and strongly consistent (using hash chains and consensus on the order of operations).

## Blockchain
An append-only hash chain, which creates tamper-evident log of transactions.

## Consensus
There is still a problem with forking of the chain (BFT protocols can achieve consensus if `n >= 3f + 1`, where `f` is the number of faulty nodes. A novel idea of bit-coin is that rather than using nodes (IP addresses) for consensus, use **work** (CPU time/electricity). In other words, the system is secure as long as the number of honest nodes have more CPU power than the dishonest ones.

**Proof-of-work**: cryptographic "proof" that certain amount of CPU work was performed

Since generating a new transaction/block requires "proof of work", forking requires a large amount of malicious work. Correct nodes also accept the longest chain, which means older blocks are safer.

## Work
Use hashing to determine work by taking advantage of the fact that hash functions are collision resistant. Work is done by finding a partial collision (for instance, a `m` whose hash `H(m)` has the most significant bit = 0).

In bitcoin, the work is to find a **nonce** such that  
`hash(nonce || prev_hash || block data) < target`

The target is recalculated in order to control the rate of harvesting.

## Mining
Creating a new block by showing proof of work creates bitcoin. There is a race to find the nonce and claim the block reward, at which time another race starts for the next block.

Miners are incentivized to accept the longest chain because only blocks that are created on the longest chain matter (majority).

A transaction is typically considered stable on the hash chain when it is 6 blocks deep (around 1 hour).

## Storage and Verification
TODO.

### Issues
Log of transactions grows very large. Since the log grows linearly, joining the race requires full download and verification of the log. 