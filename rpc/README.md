# Summary
Remote procedure calls (RPCs) allow easy-to-program network communication that makes client-server communication transparent: communication appears like a local procedure call. It retains the feel of writing centralized code for the programmer.

## Method
1. Client calls a stub function: `k = add(3, 5)`.
2. Stub marshals parameters to a network message: `proc: add | int: 3 | int: 5`.
3. OS Sends a network message to the server
4. Server OS receives message, sends it up to stub
5. Server stub unmarshals parameters, and calls server function.
6. Server function runs and returns a value
7. Server stub marshals the return value, sends the message: `Result | int: 8`.
8. Server OS sends the reply back across the network
9. Client OS receives the reply and passes up to stub
10. Client stub unmarshals return value, returns to client

## Server Stub
### Dispatcher
Receives the client's RPC request and identifies the appropriate server-side method to invoke.

### Skeleton
Unmarshals parameters to server-native types, calls the local server procedure, and marshals the resonse.

## Failures
### At-Least-Once Scheme
This is the simplest scheme for handling failures.

1. Client stub waits for a response.
2. If no response arrives after a timeout period, then client stub re-sends the request.
3. Repeat a few times, then return an error if it still fails.

But, this method can still have bad side effects, like duplication. This method is *only* acceptable if there are:

* Read-only operations with no side effects
* Idempotent and commutative functions 
* If the application has its own functionality to cope with duplication and reordering of RPCs


### At-Most-Once Scheme
Server detects duplicate requests and returns the previous reply instead of re-running the handler.  Duplicate requests are able to be detected because the client includes a unique transaction ID (random integer) with each one of its RPC requests, and uses the same id for retransmitted requests

#### Discarding Server State
In order to prune the *seen* and *old* arrays, the client includes "seen all replies up to `X`" with every RPC.

#### Concurrent Requests
Add a pending flag per executing RPC, so that the server waits for procedure to finish before responding to duplicate requests.

#### Server Crash
Need to write *old* and *seen* tables to disk, otherwise the server can accept duplicate requests.

### Exactly-Once Scheme
* Need retransmission of at least once scheme
* Needs the duplicate filtering of the at-most-once scheme
* Needs to survive crashes by logging completed RPCs to disk
* Similar to 2PC