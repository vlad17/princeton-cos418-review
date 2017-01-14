# Summary

Systems that represent distributed state can achieve varying degrees of consistency.

Chain of consistency model strength (and implications).

`Strong => Sequential => Causal => Eventual`

Theoretical model: multiple peers issue a sequence of read/write commands (operations). The associated state, for simplicity, contains key-value pairs. The view that each peer has on the system state at any point along their sequence of commands has guarantees depending on the type of consistency model our system has. A "view" is a theoretical construct. It's what the peer would see if we halted all other peers and the per in question issued a bunch of reads.

## Strong Consistency

A system is strongly consistent if at any given time along the asynchronous, simulatenous execution of multiple peer's commands, they all have the same view.

_Equivalent characterization_: every operation is given a unique time stamp, offering a global ordering for all operations, across all peers.

Also called **linearizability**

**FLP Theorem**: Even if system state is just one bit and we have a reliable network, for any deterministic algorithm reliant on asynchronous communication it is possible that no consensus will be reached (i.e., a single operation wouldn't ever be visible).

Algorithms for strong consistency (and additional assumptions):

* Multicast - no failure, reliable+synchronous network
* Paxos - random timers else livelocks, **fail/stop**, unreliable+async network (progress if majority working)
* Raft - same as Paxos but random timers part of algorithm
* Practical BFT - **Byzantine** server and client failures, unreliable+async network, valid if `>2/3` of servers work.

### Paxos (TODO: summary here, move to own folder)

**Multi-Paxos**

### Raft (TODO: summary here, move to own folder)

### Practical BFT (TODO: summary here, move to own folder)

## Sequential Consistency

## Causal Consistency

### COPS (TODO: summary here, move to own folder)

## Eventual Consistency

CAP Theorem

### Implementations

#### Bayou (TODO: summary here, move to own folder)

#### Dynamo (TODO: summary here, move to own folder)

## Other Distributed State Implementations

No exact guarnatees offered; only probibalistic.

### Chord (TODO: summary here, move to own folder)
