# Summary

Paxos is a distributed consensus algorithm.

## Paxos

Phase 1

Proposer (stores proposal counter `p`, has value `v` to offer, own `id`)

0. Increment `p`
0. Send `Prepare(p, v)`

Acceptor (stores `n`, highest accepted proposal, `(na, va)` accepted round number and value)

0. Receive `Prepare(p, v)` from proposer `prep-id`
0. if `(p, prep-id) > n`, set `n = (p, prep-id)`. If `(na, va)` undefined, set to `(n, v)`. Reply `Promise(p, na, va)` back to `prep-id`.
0. else, if `p <= n`, reply `prepare-fail` to `prep-id`

Phase 2

Proposer, upon receiving `Promise(p, ..., ...)` from majority of acceptors:

0. Choose value `va` with highest `na` from all `Promise(p, n, v)` received
0. Send `Accept!(p, va)` to all acceptors

Acceptor

0. Receive `Accept!(p, v)` from `prep-id`.
0. If `(p, prep-id) > n`, set `na = n = (p, prep-id)` and `va = v`. Then notify learners.

(Phase 3) If a learner ever receives a quorum of accepted values, it makes the decision to accept that value for consensus.

Quorum for basic paxos is majority of `N` nodes, `floor(N/2) + 1`

### Paxos safety

![paxos majority overlap](/paxos/safe.png)

### Leader elections

Have a leader-election round (like in raft): once elected, a leader makes `accept` requests directly (straight to phase 2).

A leader has a term, given by the lease recognized during the election. During the lease, followers (acceptors) will accept a current leader.

In case of a leader being partitioned, the re-elected leader may coincide with the old one (who still thinks it's his term). The re-elected leader waits until the lease ends before beginning `accept` proposals: **split-brain** can occur. However, there can only be at most one leader with an active leader lease.

## Detailed Explanation

Examples from [Wikipedia page](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29#Basic_Paxos).

Clean run:
```
Client   Proposer      Acceptor     Learner
   |         |          |  |  |       |  |
   X-------->|          |  |  |       |  |  Request
   |         X--------->|->|->|       |  |  Prepare(1, v)
   |         |<---------X--X--X       |  |  Promise(1, v) x3
   |         X--------->|->|->|       |  |  Accept!(1, v)
   |         |<---------X--X--X------>|->|  Accepted(1, v)
   |<---------------------------------X--X  Response
   |         |          |  |  |       |  |
```

Livelock without random timeouts:

![paxos livelock](/paxos/livelock.png)
