# Summary

Two-Phase Commit is an atomic commit protocol with a goal of distributed agreement on some action with fault tolerance.

## Goals of 2PC
**Safety**:  
(1) If one person commits, *no one* aborts.  
(2) If one person aborts, *no one* commits.
  
**Liveness**:  
(1) If there are no failures and A and B can commit, then the action should commit.  
(2) If there are failures, reach a conclusion as soon as possible.

## Commit Request Phase
(1) The coordinator sends a query to commit message to all servers and waits until it has received a reply from all servers.

(2) Each servers replies with an agreement message (Yes, to commit), if the server's is ready, or an abort message (No, not to commit), if the server experiences a failure that will make it impossible to commit.

## Commit Phase
If the coordinator received an agreement message from all cohorts during the commit-request phase:

(1) The coordinator sends a commit message to all the servers.  
(2) Each server completes the operation, and releases all the locks and resources held during the transaction.  
(3) Each server sends an acknowledgment to the coordinator.  
(4) The coordinator completes the transaction when all acknowledgments have been received.

### Failures
If any server votes "No" during the commit-request phase (or the coordinator's timeout expires):

(1) The coordinator sends a rollback message to all the servers.  
(2) Servers release the resources and locks held during the transaction.  
(3) Each server sends an acknowledgement to the coordinator.  
(4) The coordinator responds to the client when all acknowledgements have been received.

## Write-Ahead Log
The write ahead log is used to record "commit" or "yes" to disk, so that if any node crashed, they can reason about their state.

### Recovery

**TC**: If no commit record on log, abort.  
**Servers**: If no "yes" record on disk, abort, as TC couldn't have committed. If there is a "yes" record, finished executing.


## Example
```
C = Client
TC = Transaction Coordinator
A, B = Servers

1) 	C -> TC: Transfer 100 to A
2) 	TC -> A, B: Prepare
3) 	A,B -> TC: "Yes" or "No"
4) 	- If A and B say "Yes", TC -> A,B: "Commit"  
	- If A or B say "No", TC -> A,B: "Abort"
5) A,B commit on receipt of message.
```

## Issues
The greatest disadvantage of the two-phase commit protocol is that it is a blocking protocol. If the coordinator fails permanently, some servers will never resolve their transactions: After a server has sent an agreement message to the coordinator, it will block until a commit or rollback is received.

## Failure Examples
TODO: Participant fails before sending response

TODO: Participant fails after sending vote

TODO: Participant's vote was lost during transmission

TODO: Coordinator fails before sending prepare

TODO: Coordinator fails after sending prepare

TODO: Coordinator fails after receiving votes

TODO: Coordinator fails after sending decision

TODO: Blocking if coordinator fails