# Summary

Spark provides an interface for programming entire clusters with implicit data parallelism and fault-tolerance.

It provides distributed task dispatching, scheduling, and basic I/O functionalities, exposed through an application programming interface centered on the RDD abstraction.

**Resilient distributed datasets** are a read-only multiset of data items distributed over a cluster of machines.

RDDs are immutable and their operations are lazy; fault-tolerance is achieved by keeping track of the "lineage" of each RDD (the sequence of operations that produced it) so that it can be reconstructed in the case of data loss.