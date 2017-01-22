# Summary
**Primary-backup** is a highly reliable service with the goal of making stateful replicas fault tolerant. It allows replicas to provide continuous service despite net and machine failures.

## Goals
* Provide a highly reliable service that despite some server/network failures, is able to continue operation after failure  
* Servers should behave just like a single, more reliable server
* Distributed **all-or-nothing** atomicity - an operation should be executed on all replicas, or none at all

## Approach
* Nominate one server as the primary (should have only *one primary* at a time), and call the other(s) the backup server  
* Clients send all operations (get, put) to current primary, who orders the clients' operations
* A **non-backup** must reject forwarded requests
* A backup accepts forwarded requests only if they are in its idea of the current view

### Normal Operation
1. Primary logs the operation locally
2. Primary sends operation to backup and waits for an ack
3. Backup adds operation to its log or applies operation, then acks to primary
4. Primary performs operation and acks to the client 

## View Server
A **view server** decides who is primary and who is backup. Clients and servers depend on the view server.

Each replica periodically pings the view server, which declares the replica **dead** if it missed `N` pings in a row or alive after a single ping.

A **view** consists of the (`view #`, `primary server idx`, `backup server idx`). 

### Agreeing on the Current View
Any number of servers can ping the view server. The view # is included in RPCs between all servers.

Servers that have a view that is more up-to-date view than that of a message they are receiving will reject it and reply with their belief of the current view.

### Transitions
To ensure a new primary has an up-to-date state, only promote a previous backup. If a previously idle server is the primary, it will have old state.

A view server knows whether a backup is up to date by sending a **view-change** message to all. The primary will ack the new view once the backup is up-to-date. The view server stays with the current view until the ack, even if the primary has appeared to fail.

## State Transfer
A *new* backup (in view `i` but not in view `i-1`) gets the current state through the primary. This occurs through state transfer via snapshot. If an operation occurred before the transfer, the transfer must reflect the operation. If the operation occurred after the transfer, the primary forwards the operation to the backup after the state transfer finishes.

## Examples
### Split Brain
**Split brain** can occur, where the view of who is primary and backup differs among servers.

![primary-backup](/primary-backup/split_brain.png)

### Stale Server View
In this example, the client has the updated view and tries to contact the primary, who doesn't have the updated view. `S2` can infer from the client that it has a stale view, and updates its own view.
![primary-backup](/primary-backup/ex1_1.png)  
![primary-backup](/primary-backup/ex1_2.png)  

### Stale Client and Primary View
In this example, the client has the old view and tries to contact the *old* primary, who also has the old view. When the primary forwards the command to `S2`, `S2` rejects the request. The client's request can be rejected and the client can infer that it has a stale view, and updates its own view.

![primary-backup](/primary-backup/ex2_1.png)  
![primary-backup](/primary-backup/ex2_2.png)  

### Stale View in System
In this example, the client and both servers have a stale view. The client's request still proceeds as normal.  
![primary-backup](/primary-backup/ex3.png)  

