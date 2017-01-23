# Summary

Stream operations throught data flow construction. Stateless operations can be trivially parallelized. Stateful aggregation (by key), can be parallelized through **hash-joins**.

Data flow can be **tuple-by-tuple** or **micro-batch**. Latter is higher throughput, higher latency.

Semantics offered differently by differnet frameworks.

**Apache Storm** offers at-least once semantics: spout waits for ack when sending a tuple, resends if timout. Application must handle out-of-order and re-sent tuples. Streams of tuples is data. Achieves fault tolerance through record acknowledgment.

**Apache Spark**. Easy windowing through microbatches on top of classical batch Spark. Splits streams into micro batches of a few seconds each. Emit each micro-batch result as an RDD. Achieves fault tolerance through these micro-batches and RDD.

**Google Cloud Dataflow** (also _Google Millwheel_). Record intermediate results in a high-throughput distributed state store. Achieves fault tolerance through transactional updates into a log. On failure, replay log.

**Apache Flink** uses distributed snapshotting, based on Chandy-Lamport. Achieves fault tolerance by recovering from latest snapshot.
