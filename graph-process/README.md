# Summary

Using map-reduce for ML graph algorithms: solves the problem of writing low-level parallelism code but through a crude abstraction. Data parallelism cannot exploit model parallelism. Bulk-synchronous parallel graph iterations hit disk penalty between iterations, and have needless recompute of inactive nodes.

**GraphLab**: use smart, fast partitioning of th graph so that every machine is responsible for local vertices. Machine in charge of local scheduling. Remote machines request inter-machine adjacency information from other machines. Can tune consistency: fully asynchronous, race-free vertex updating until convergence (may take long). Consistency model that's equivalent to synchronous but still better than BSP is **color-step** based. Also offers **distributed locking** that's made faster with pipelining.

**Chandy-Lamport** checkpointing. Initiator atomically receives a marker. If a non-red node receives a marker, do atomically:

0. Turn red
0. Record state 
0. Send markers to all neighbors

Then red nodes start recording incoming messages (these will be "in flight" in the snapshot).

Under color-step consistency, the above update function gives asynchronous snapshotting (and therefore easy fault tolerance). Assumes all messages arrive in tact and channels are FIFO, bidirectional.
