# Summary

Dynamo builds a **horizontally-scalable** system that seeks to achieve high availability and low-latency response.

## Techniques From Chord

Use virtual nodes and consistent hashing for a P2P-based distributed hash table, with low load rebalancing requirements per node in case of failure.

## Distinctions from Chord

All machines are local and trusted.

**Gossip**: P2P exchanges of known virtual node IDs. This means that over time all machines get to know the machines responsible for a given range of key values. Linear amount of state (unlike chord), but enables zero-hop DHT, speeds up response times.

## Consistency and Fault Tolerance

Every key has a virtual node, **preference list** of size `N` from `size` Chord-like successors. This is the replication bandwidth, _sort of_. `N < size` successors is chosen to have distinct physical nodes.

Weak, eventual consistency: in case of a partition, writes are still allowed to **sloppy quorums**. Risk returning _stale data_ or having _write conflicts_ upon partion heal. For `put`, suffices to have `W <= N` writers. For `get`, suffices to have `R <= N` writers.

**Hinted handoff**: even if `put` doesn't get `W` replies, coordinator tries successors past `N` to agree, telling them to update the original recipient as soon as possible.

Preference list contains cross-data-center nodes for (eventual) reliability.

Sloppy quorums guarnatee that if `R + W > N` and we have no failures then we'll have sequential consistency. With node failures, sloppy quorums can bleed to stale successors outside the preference list.

## Conflict Resolution

Suppose a coordinator receives two conflicting reads - can it do better than just tell the client?

Use sort-of **vector clock** (index is the IDs of the coordinator nodes, and the counter is the number of `put` requests they've received). Local node storage keeps version vector for associated data key-pair.

If the same key has two rows with vectors `v1 < v2`, can forget `v1` since coorinator originating data with version `v2` must've seen `v1`'s changes.

This means client must maintain causal consistency - a `get` served by an advanced node comes with the version of the key. A following `put` (which might be served by an older node) must be accompanied with the context.

If two versions of the data have `v1 || v2`, client must reconcile the data in a client-aware merge.

Version vectors drop timestamps of oldest nodes when too long to keep size short. This means that some merges that could've been automatically dealt with are now surfaced to client.

## Eventual Durability

Use **replica synchronization** where nodes pass around the keys and values they hold. Detect symmetric diffence of keys held in machines by using Merkel trees (heierichal hashing): exchange root first: if same, then replicas have same content. If no match, compare children. Descend into branches that conflict.

# Detailed Explanation

## Merkel Tree Synchronization

![merkel](/dynamo/merkel-recon.png)

## Sloppy Quorum Sizes

![sloppy](/dynamo/nrw.png)
