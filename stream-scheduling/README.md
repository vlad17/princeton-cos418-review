# Summary

Stream operations throught data flow construction. Stateless operations can be trivially parallelized. Stateful aggregation (by key), can be parallelized through **hash-joins**.

Data flow can be **tuple-by-tuple** or **micro-batch**. Latter is higher throughput, higher latency.

Semantics offered differently by differnet frameworks.

**Apache Storm** offers at-least once semantics: spout waits for ack when sending a tuple, resends if timout. Application must handle out-of-order and re-sent tuples.

**Apache Spark**. Easy windowing through microbatches on top of classical batch Spark.

**Google Cloud Dataflow** (also _Google Millwheel_). Record intermediate results in a high-throughput distributed state store.

**Apache Flink** uses distributed snapshotting, based on Chandy-Lamport.
