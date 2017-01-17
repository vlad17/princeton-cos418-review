# Summary

**SPORC** allows for group Collaboration using Untrusted Cloud Resources. It allows real-time/concurrent collaboration (editing of shared data) and offline work (disconnected operation), and is used with untrusted servers that cannot read user data and cannot tamper without risking detection. SPORC illustrates the complementary benefits of operational transformation (OT) and fork* consistency.

See conflict_resolution for details on operational transform.

## Malicious Server
Servers play a very limited role: only for storage and ordering messages, as all messages are encrypted. But, digital signatures aren't enough, because servers can equivocate by forking clients.

### Fork* Consistency
In the **fork consistency** model, if the server causes the views of two clients to diverge, the clients must either never see each others’ subsequent updates or else identify the server as faulty. A correct server must have the total order of writes. However, faulty server may show different version to different clients.

In an honest server, SPORC achieves linearizability. In a malicious server, the clients can detect equivocation after exchanging 2 messages by embedding a hash chain in every message.

## Algorithm (High Level)
1. Client `A` has committed entries (which are fork* consistent). It wants to a commit another operation `k`, and encrypts + signs and sends to the server.
2. When Client `B` gets this operation, it knows that if operation `k` depends on all preceding operations, then the hash chain history must be point to those operations.
3. If the hash chain histories are different, the clients can start recovery by using operational transform.

### Recovery
Recovering from a malicious fork is similar to reconciling concurrent operations in the OT framework. Upon detecting a fork, SPORC clients use OT mechanisms to replay and transform forked operations, restoring a consistent state.
