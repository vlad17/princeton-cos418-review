# Summary

Commodity PC clocks exhibit 1s skew every 2 days. Need correction from reliable server, but transmitting the correct time takes time, so reliable response will be out of date.

## Cristian's algorithm addresses staleness under some assumptions.

![Cristian's algorithm demo](/time/cristian.png)

**Berkeley algorithm** is an average of Cristian's algorithm applied to several servers.

## NTP
NTP is a reliable, accurate service. Expand availability as we go into higher **strata**:

0. High-precision time sources
0. NTP clients of stratum 0
0. Ibid, for strata 1
0. Ibid, for strata 2 (service-level)

Suppose a level `n+1` client is talking to a level `n` server. NTP synchronization:

0. Perform Cristian's algorithm 8 times with the `k`-th server (without synchronizing). Let the `i`-th interaction yield an offset `t(k, i)` and RTT `d(k, i)`.
0. For each compute the **dispersion** `s(k)=max D(k) - min D(k)` where `D(k)={d(k, i) for integers 0<=i<8}`.
0. Choose the offset `t(k*, i*)` of the minimum-RTT message `i*` of the lowest-dispersion server `k*`.

Note NTP adjusts clock gradually and monotonically by changing the rate of change.

## Logical clocks

Provide a logical time that establishes an order over all events. For every server `k`, keep a local counter `c(k)`.

For every local event, let event time be current value of `c(k)`; then increment the counter. Logical time also contains what server it came from.

Upon receipt of any message from another server `j` (where the message being sent is an event, having some value `s`), set the local clock to be `1+max(c(k), s)`.

Any event time is ordered by its value, ties broken by server ID. This guarantees strong ordering.

**Multicast consensus**. Logical clocks allow for a consensus algorithm (assuming no failures). All nodes do the following:

* Upon receiving a proposal (to change global state), broadcast with current local time.
* Upon receiving a broadcasted proposal, add it to a time-sorted queue of proposals; broadcast an ACK for that proposal to everyone.
* If the head of the queue is universally ACKed, apply it to the distributed global state machine.

## Vector Clocks

Every peer keeps a counter for the largest known time for every other peer and itself.

Upon a local event, increment own counter.

Upon receiving a message from a peer (with the peer's counters for everyone), increment own counter and update maximum time known for all peers based on peer's counters.

List of all counters at server `k` is the vector time, `V(k)`. Let `V(a) <= V(b)` if it holds entrywise.

Note the operator `<=` induces the operators `=, <`.

Vector clocks capture **causality**. If and only if an event at time `v1` could be causally related to another at time `v2` (there is a chain of events from on to the other), possible at different servers, then `v1 < v2`. If no relation (even `<=`) holds between `v1, v2`, then they are concurrent.

## Detailed Explanation

Note that vector clocks hold with inequality iff events can be causally related. This enables systems, such as Dynamo, to infer (if neither `V(a) < V(b)` nor `V(b) < V(a)` hold) that two events are concurrent (and their peers have not yet interacted). This is useful for determining if someone has received someone else's updates or not.

