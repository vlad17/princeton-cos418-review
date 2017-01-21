# Summary

Raft implements distributed consensus in a more modular way than Paxos by using a strong leader and separating the algorithm into four modules that are not fully independent but intersect minimally: **leader election**, **log replication**, **snapshotting**, and **membership changes**. The last two were not studied in this course.

## Server States
(1) **Leader**: handles all client interactions and log replication  
(2) **Follower**: completely passive, follow instructions of leader  
(3) **Candidate**: a server that is vying to become the new leader 

![state machine](/raft/state_machine.png)  
_Image source_: In Search of an Understandable Consensus Algorithm (Extended Version), Ongaro and Ousterhout 2004.  

* Every server maintains a **term** value, which is incremented when a follower turns into a candidate
* Servers start as followers

## Leader Behavior 
* Leaders send **heartbeats** (empty AppendEntries RPCs) to maintain authority over followers
* If a command is received from the client, append the entry to the local log and respond after the entry is applied to the state machine
* Forward proposals to append to the log to other followers via AppendEntriesRPC

## Follower Behavior
* On receiving heartbeat, reset the election timer
* On receiving nonempty append entries from leader, update log entries
* If an **electionTimeout** (150-300ms) passes with no heartbeat, the follower assumes the leader has crashed, converts to candidate, and starts a new election
* Reject any RPC that was sent by a stale leader


## Candidate Behavior
### Election
(1) When an election is started, increment the current term and change to candidate state.  
(2) Send RequestVote to all other servers and retry until one of three possibilities occurs  

```
	(1) Receive votes from majority of servers and become the new leader  
	(2) Receive an RPC from a valid leader (higher term)  
	(3) Election timeout elapses, and a new election is started
``` 

### Voting
* A server only grant's a vote if the candidate's log is more complete than it's own log (via higher term or higher index of same term)
* As a result, the leader will have the "most complete" log among the electing majority

**Safety** is guaranteed because there is at most one winner per term (there can't be two majorities).  
**Liveness** is guaranteed because some candidate must eventually win (probabilistically).

## Log
A log entry has three components, the **index**, the **term**, and the **command**. It is stored on stable storage. An entry is considered **committed** if it is stored on the log on a majority of servers.

**Safety**: If a leader has decided a log entry is committed, then that entry will be present in logs of all future leaders. 

### Log Matching Property
**If log entries on a different server have the same index and term, they store the same command, and all preceding entries are identical.**

### Consistency Check
Inconsistencies can result due to partitions, disconnections, network failures, etc. Leader crashes, for instance, can leave logs inconsistent (the old leader may not have fully replicated all of the entries in its log). These inconsistencies can compound over a series of leader and follower crashes. 
  
![raft inconsistencies](/raft/raft_inconsistencies.png)  
_Image source_: In Search of an Understandable Consensus Algorithm (Extended Version), Ongaro and Ousterhout 2004.     

If a follower's log doesn't match with the leader's log during an AppendEntries RPC (i.e. the terms don't match), then the leader will repair the follower's log by finding the last entry that is matching, and then sending the follower all entries from that point on.

## Normal Operation
(1) Client sends a command to the leader  
(2) Leader appends the command to its own log  
(3) Leader sends AppendEntries RPCs to followers, telling them to append the command to their log. The Leader retries these RPCs until they succeed (response from followers)  
(4) Once the entry exists on a majority of servers, the leader applies the command to its state machine, and replies to the client.  
(5) Leader piggybacks this commitment to other followers in later AppendEntries  
(6) Followers apply command to their state machine. 

## Configuration Changes
Configuration changes are done using a 2-phase approach via **joint consensus**. There must be joint consensus in the intermediate phase (between the servers in the old configuration and the new configuration). In other words the majorities of two different configurations overlap during transitions, which allows the cluster to continue operating normally during configuration changes.

Configuration change is just a log entry and is applied immeidately upon receipt.

## Discussion
* Ensures **exactly-once semantics** even with leader failure.
* Strong leader because (1) log entries flow only from leader to other servers, and (2) leader is selected from a limited set so it's log is up to date
* Leader election is done through randomized timers
* Distributed consensus algorithm for managing a replicated log
