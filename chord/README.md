# Summary

Chord is a scalable peer-to-peer distributed lookup service. It provides support for just one operation: given a key, it maps the key onto a node. Data location can be easily implemented on top of Chord by associating a key with each data
item, and storing the key/data item pair at the node to which the key maps. It is efficient (`O(log N)` messages per lookup) and scalable (`O(log N)` state per node), where N is the total number of servers. The total expected routing time is `O(log N)`. It is also robust, surviving massive failures.

## Peer-to-Peer (P2P)
### Properties
* A distributed system architecture with no centralized control
* Nodes are roughly symmetric in function
* Large number of unreliable nodes
* Example: BitTorrent

### Advantages
* Absence of a centralized server means there is easy deployment
* Less chance of service overload
* Avoidance of a single point of failure

### Disadvantages
* High latency and limited bandwidth between peers
* User computers are less reliable than managed servers
* Lack of trust in peers' correct behavior
* Security problems are difficult

## Distributed Hash Table
Decentralized distributed system that provides a lookup service similar to a hash table - constant-time insertion and lookup. The application and data storage is distributed over many nodes.

![state machine](/chord/dht.png)

## Lookup
![state machine](/chord/lookup.png)

**Centralized Lookup** (Napster): a simple database stores the location of data. However, requires `O(N)` state and is a single point of failure.

**Flooded Queries** (Gnutella): client floods peers for location of data. Robust, but `O(N = number of peers)` messages per lookup.


## Chord Algorithm

### Consistent Hashing
**Key identifier**: SHA-1(key)  
**Node identifier**: SHA-1(IP address)

This hash distributes the identifiers randomly in hash space.

![state machine](/chord/consistent_hashing.png)  

Each node also has pointers to its successor, the node with the next-higher ID. 

### Finger Tables
Keep a table of pointers to successors. Finger `i` at node `n` points to the successor of node `n + 2^i`. This is essentially a binary lookup tree rooted at every node. **Every** node has its own finger table and acts as a root, so there's no root hotspot and no single point of failure.

### Node Join
The following examples are for adding `N36`, where the initial state is `N25 -> N40`. `N40` has keys `K26`, `K30` and `K38`.

1. `Lookup(36)` and get `N40`.
2. `N36` sets its own successor pointer to `N40`.
3. Copy keys `K26` to `K36` from `N40` to `N36`.
4. Notify Messages: `N40` notifies `N25` that it's predecessor is `N36` 
5. Finger pointers updated in the background

### Successor Lists
Each node stores a list of its `r` immediate successors, so after failure, the first live succesor is known.

### Chord Lookup
```
Lookup(key-id)
	look in local finger table and successor-list
		for highest n: my-id < n < key-id
	if n exists
		call Lookup(key-id) on node n	 // next hop
		if call failed,
			remove n from finger table and/or successor list
			return Lookup(key-id)
	else 
		return my successor	 // done
```

## Application: DHash DHT
* Chord used to build key/value storage.
* Replicates blocks for availability by storing replicas on successors
* Authenticates block contents
* Replicas are easy to find if successor fails (via sucessor list)
* Hashed node IDs ensure independent failure

## Discussion
* Quick lookup in large systems
* Low variation in lookup costs
* Robust despite massive failure
* With `N^2` virtual nodes, a node failure only increases the load on each node by `1/N` in expectation, compared to the case of `N` virtual nodes (where if a node fails, all its keys goes to its successor)



