# Summary

A **distributed transaction** is a database transaction in which two or more network hosts are involved. Google's Spanner is a Globally Distributed Database.

## Spanner

Spanner manages global replication across datacenters.

Unit of replicated data is a _tablet_, or a set of rows.

### Tablet Operations

A logical tablet is replicated across datacenters, and replicas are kept consistent with Paxos.

Transactions _within_ a tablet only need to use an internal 2PL protocol managed on the tablet's Paxos leader. 2PL used over OCC because long-running transaction degrade OCC perf in presence of contention.

### Serializability Guarantee

**TrueTime**: a global "wall-clock time" with bounded uncertainty: Google maintains a service that gives exact time within a bounded interval: allows processors to know that a true time has certainly not passed or certainly passed.

Serializability guaranteed with a 2PL system iff we can (1) assign to each transaction `i` (`T(i)`) a timestamp in between the true time of all lock acquisitions amongst involved tablet groups and corresponding lock releases and (2) if `T(i)` is assigned a timestamp (`s(i)`) and the true start of `T(i)` is less than the assigned `s(j)` for some `j`, then `j`'s writes must be invisible to `T(i)`. Then a transaction can't see the effects of concurrent or future transactions. However, every transaction that commited before `s(i)` will have its effects visible to `T(i)`, guaranteed by 2PL and Paxos.

For this reason, we need **commit-wait**. This means that Spanner has to wait until `s(i)` _definately passed_ before enacting the commit corresponding to `T(i)` in the local Paxos group. This makes a chain of inequalities provable to ensure the above property (see the paper).

### Cross-Datacenter Transactions

In the case the transaction has to span multiple Paxos groups, we need a 2-phase commit: one group's leader becomes the coordinator.

0. Client acquires read locks from all leaders, reads data.
0. Client performs local writes with data from all leaders from cross-client transacion.
0. Release read locks (at least in theory can be done here)
0. Choose a coordinator, send every leader the writes and coordinator id.
0. Coordinator waits for prepare from all leaders (a bunch of local paxos rounds with prepared writes)
0. Upon receiving all prepare timestamps, choose highest including max with local timestamp), perform local commit in coordinator's local Paxos group (coordinator never does a prepare round) (this commit could be an abort upon timeout from non-coordinator prepares).
0. Commit-wait
0. Apply to state machine, tell others to commit (who in turn do a local Paxos commit and unblocked apply).
0. Release write locks

![spanner 2pc](/spanner/2pc.png)

In other words: layer 2PC over Paxos groups. Commit wait combats eager 2PL read-interleaving in the interval between [read locks released, write locks released]. An equivalent implementation with a single global Paxos would (on top of preventing disjoint transactions from running) make that interval empty. 

Note the state machine's timestamp will always lag behind true time. We just advance its recorded true time snapshot when applying to it (`t_safe^Paxos` in paper).

### Read-only Optimizations.

Maintain `t_safe=min(t_safe^Paxos, t_safe^TM)`, `t_safe^TM` is transaction manager time, and is less than the minimum prepare timestamp live for that Paxos group (nothing can be commited before this time).

Maintaining older versions of the state machine enables snaphsot reads.

Condition 4.1.3 below is making sure that the read timestamp is less than or equal to `t_safe`.

![spanner transaction table](/spanner/spanner-ro.png)

Note transaction reads may require `t_safe` to advance sufficiently, especially for cross-tablet reads.

As long as the assigned time is at least the maximum of all involved tablet groups' latest true commit time, the read will be externally consistent (thanks, true time!). However, it's faster to just use `TT.now().latest`, which guarantees this property rather than checking with multiple Paxos leaders.


