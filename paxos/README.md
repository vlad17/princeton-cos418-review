# Summary

Paxos is a distributed consensus algorithm.

## Paxos

Phase 1

Proposer (stores proposal counter `p`, has value `v` to offer, own `id`)

0. Increment `p`
0. Send `prepare(p, id, v)`

Acceptor (stores `n`, highest accepted proposal, `(na, va)` accepted round number and value)

0. Receive `prepare(p, prep-id, v)`
0. if `(p, prep-id) > n`, set `n = (p, prep-id)`. If `(na, va)` undefined, set to `(n, v)`. Reply `promise(p, na, va)` back to `prep-id`.
0. else, if `p <= n`, reply `prepare-fail` to `prep-id`

Phase 2

Proposer, upon receiving `promise(p, ..., ...)` from majority of acceptors:

0. Choose value `va` with highest `na` from all `promise(p, n, v)` received
0. Send `accept(p, id, va)`

Acceptor

0. Receive `accept(p, prep-id, v)`
0. If `(p, prep-id) > n`, set `na = n = (p, prep-id`) and `va = v`. Then notify learners.

If a learner ever receives a quorum of accepted values, it makes the decision to accept that value for consensus.

### Paxos safety

![paxos majority overlap](/paxos/safe.png)

### Leader elections

Have a leader-election round (like in raft): once elected, a leader makes `accept` requests directly (straight to phase 2).

A leader has a term, given by the lease recognized during the election. During the lease, followers (acceptors) will accept a current leader.

In case of a leader being partitioned, the re-elected leader may coincide with the old one (who still thinks it's his term). The re-elected leader waits until the lease ends before beginning `accept` proposals: **split-brain** can occur.

## Detailed Explanation

[Wikipedia page with examples](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29#Basic_Paxos)

Livelock without random timeouts:

![paxos livelock](/paxos/livelock.png)
