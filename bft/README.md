# Summary

Byzantine faults are different from fail/stop: faulty processors and clients can send arbitrary data to anyone, including replaying stale signed messages they received from others.

`f` failures can be tolerated with `3f+1` replicas, _the best you can do_. Implemented in Practical BFT.

Client protocol: wait for `f+1` identical replies, in which case at least 1 is non-faulty.

Proposing changes in BFT. Primary-backup style. At every moment of operation, there exists a view agreed-upon by at least `2f+1` replicas (view = who is the leader).

Change proposals by leader:

0. Leader sends signed change
0. Replicas verify (1) correct leader, (2) change has correct `seq` number.
0. Replicas broadcast signed acceptance of change (prepared replicas)
0. Each replica collects everyone's signature. Once it has `2f+1`, it broadcasts a commit certificate
0. With enough (`2f+1`) commit certificates received, each replica commits.
0. Replica sends confirmation directly back to client

Note faulty leader can avoid progress, but not make faulty progress (assuming client is signed).

A view change can be proposed by anyone, requires an election, `2f+1` votes, and broadcasting the new view (with all the voter's signatures).

`2f+1` quorums intersect with `f+1` nodes, so at least 1 non-faulty replica maintains correctness at all times.

Liveness requires view changes can't come too often. Can only GC change log upon agreement (via a proposal), not after each commit.
