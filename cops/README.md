# Summary
**COPS** is a key-value store that delivers a causal consistency model across the wide-area. A
key contribution of COPS is its scalability, which can enforce causal dependencies between keys stored across an entire cluster.

## Overview
Want to server requests quickly in wide-area storage.

**ALPS**:  

* Availability  
* Low Latency
* Partition Tolerance
* Scalability

To achieve ALPS, COPS serves operations locally, and replicates in the background. It partitions the keyspace onto many nodes, and controls replication with dependencies.

A client library exposes a put/get interface to its clients and ensures operations are properly labeled with causal dependencies. A key-value store replicates data between clusters, ensures writes are committed in their local cluster only
after their dependencies have been satisfied, and stores multiple versions of each key along with dependency metadata.


### Scalability
Scalability is achieved by having dependency metadata explicitly capture causality. Distributed verifications should replace single serialization (delay exposing replicated puts until all dependencies are satisfied in the datacenter).

### Dependencies
Dependencies are explicit metadata on values. The library tracks and attaches them to put_afters. Each key-value pair has associated metadata, which contains a version number.

Thus, a **put_after** call contains the key, the value, and the dependencies.

**Nearest dependencies** are used in order to minimize the number of dependency checks.

![state machine](/cops/dependencies.png)  
_Image source_: Don’t Settle for Eventual:
Scalable Causal Consistency for Wide-Area Storage with COPS, Lloyd et al., 2004. 

## Reads
Reads are satisfied in the local cluster. Clients call the get library function with the appropriate context; the library in turn issues a read to the node responsible for the key in the local cluster.

## Writes
All writes in COPS first go to the client’s local cluster and then propagate asynchronously to remote clusters. The key-value store exports a single API call to provide both operations. 

The **put_after** operation ensures that `val` is committed to each cluster only after all of the entries in its dependency list have been written. In the client’s local cluster, this property holds automatically, as the local store provides linearizability.

After a write commits locally, the primary storage node asynchronously replicates that write to its equivalent nodes in different clusters. In these clusters, the remote nodes can commit an update only after its dependencies have been committed. The remote nodes check this by issuing a dependency check (`dep_check`), which blocks until the dependencies are satisfied.