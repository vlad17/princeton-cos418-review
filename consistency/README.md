# Summary

Systems that represent distributed state can achieve varying degrees of consistency.

Chain of consistency model strength (and implications).

`Strong => Sequential => Causal => Eventual`

Theoretical model: multiple peers issue a sequence of read/write commands (operations). The associated state, for simplicity, contains key-value pairs. The view that each peer has on the system state at any point along their sequence of commands has guarantees depending on the type of consistency model our system has. A "view" is a theoretical construct. It's what the peer would see if we halted all other peers and the per in question issued a bunch of reads.

## Strong Consistency

A system is strongly consistent if for any realization, at any given time along the asynchronous, simulatenous execution of multiple peer's commands, they all have the same view.

_Equivalent characterization_: every operation is given a unique time stamp, offering a global ordering for all operations, across all peers.

Also called **linearizability**

**FLP Theorem**: Even if system state is just one bit and we have a reliable network, for any deterministic algorithm reliant on asynchronous communication it is possible that no consensus will be reached (i.e., a single operation wouldn't ever be visible).

Algorithms for strong consistency (and additional assumptions):

* Multicast - no failure, reliable+synchronous network
* Paxos - random timers else livelocks, **fail/stop**, unreliable+async network (progress if majority working)
* Raft - same as Paxos but random timers part of algorithm
* Practical BFT - **Byzantine** server and client failures, unreliable+async network, valid if `>2/3` of servers work.

## Sequential Consistency

A global ordering of the operations exists, and it respects the local ordering of every peer's own operations. However, this ordering need not be realizable (operations can be re-ordered: two views need not agree at the same time).

## Causal Consistency

Any two writes that _could_ be causally related are properly sequenced. If every operation was given a vector clock value (where messages are propogated by reading the state that someone wrote), then vector clock order would always be respected by a causally consistent system.

## Eventual Consistency

**CAP Theorem**: Can have at most 2 of the three: Linearizability, Availability, Partition-tolerance.

Eventual systems: AP, with updates guarnateed to propogate in finite time, if no new changes.

## Chain Replication

![chain replication image](/consistency/chain.png)

In read-heavy workloads, reads can be answered at the replicas in the middle. If a write is being propogated down the chain, then the requested key can be dirty, in which case the exact tail version is requested.

# Detailed Explanation

## Sequential but not Strong

Example realization of sequentially consistent but not strong consistent system uin operation.

```
time    0         1         2         3
Peer 1: W (a) x=1           R (b) x=1 R (e) x=2
Peer 2:           W (c) x=2 
Peer 3:                     R (d) x=2
```

Note in the above example it would be impossible for any peer to read `2` then `1`. Global order respecting local order (and therefore evidence of sequential consistency) is given by letters.

## Causal but not Sequential


Example of causally but not sequentially consistent run:

```
time    0     1     2     3
Peer 1: W x=1         
Peer 2:       W x=2 
Peer 3:             R x=2 R x=1
Peer 4:             R x=1 R x=2
```

From the write/read pairs we can extract the implicit messages passed through the system. In the format `peer-time`, we notice we have the following messages:
`1-0 -> 3-4`, 
`2-0 -> 3-3`, 
`1-0 -> 4-3`, 
`2-0 -> 4-4`. For every peer, there exists a consistent DAG respecting the following messages (every read must be incident from a corresponding write, every local ordering must be obeyed).

In particular, for Peer 3:
```
1: W x=1               
2:   \                  W x=2
      \                   \
3:    (received) - R x=2 -(received) - R x=1
```
Thus, there's a valid assignment of vector clocks to the above realization.

There's also no sequential consistency because no single global order will respect 3's local order `R x=2, R x=1` and 4's local order `R x=2, R x=1` at the same time.

From the above we also see that if Peer 2 had a `R x=1` before the write `W x=2`, at time `0.5`, no causal consistency would be contained. In particular, the following messages must be respected (now with the additional link `1-0 -> 2-0.5`).

```

1: W x=1 -------------------------------
     \                                  \
2:   (received) - R x=1 - W x=2          ------
                            \                  \
3:                          (received) - R x=2 -??? - R x=1
```
