# Summary

Bayou is an eventual-consistency replicated state machined designed remote (fast) operation even under loss of connectivity.

Provides a different abstraction over replicated state than classical strong consensus (arbitrary bits and deterministic operations).

Let application handle merge behavior: create notion of determinstic **updates** to state which change behavior based on possible conflicts. If applied in same order, they yield the same result

Total order on (_uncommited_) updates determined by Lamport clocks (last two entries of Bayou record).

Updates have to have **rollbacks** so that they can be undone if earlier updates (not yet applied are found).

Synchronization between two remote devices: exchange log indices, re-sort logs when everyone has everything.

Can't commit based on Lamport total order: have no confirmation that an earlier write will come, forcing a rollback. Designate a **primary** server that hands out **commit sequence numbers**. Once its earliest _uncommited_ update is given a CSN (in montonically increasing order), that commit is propogated via the same synchronization.

Notes:

* Tentative order is NOT commit order in Bayou
* Tentative order on machines CAN conflict, even after sync!
* Primary must preserve local order of commits (commits, filtered by the same node ID, will have increasing timestamps)
* Can discard all commited CSNs up as long as you snapshot database
* CSN syncs can be done quickly by exchanging CSN indices.

Use **version vectors** (each replica keeps counter for number of updates from each origin lepica that it has in its tentative log). Requires care for configuration changes (for trimming version vectors: need records in other servers indicating a server joined/left). Solution: identifier is timestamp of joining.

# Detalied Explanation

## Tentative Order Conflicts after Sync

3 replicas `A,B,C` and primary `P`. Logs are in format `(CSN, Time, ID)`, where an unassigned `CSN` is `-`.

```
t=0 C updates
t=1 B updates
t=2 A updates

t=2 state
A: (-, 2, A)
B: (-, 1, B)
C: (-, 0, C)
P:
```

```
t=3 A-B sync

t=3 state
A: (-, 1, B) (-, 2, A)
B: (-, 1, B) (-, 2, A)
C: (-, 0, C)
P:
```

```
t=4 B-C sync

t=3 state
A: (-, 1, B) (-, 2, A)
B: (-, 0, C) (-, 1, B) (-, 2, A)
C: (-, 0, C) (-, 1, B) (-, 2, A)
P:
```

Note that `A,B` synced but disagree.

## Tentative Order May Disagree with Commit Order

```
t=0 A updates
t=1 B updates

t=1 state
A: (-, 0, A)
B: (-, 1, B)
C:          
P:          
```

```
t=2 B-P sync
t=3 B-C sync

t=3 state
A: (-, 0, A)
B: (-, 1, B)
C: (-, 1, B)
P: (-, 1, B)
```

```
t=4 A-C sync
t=5 P commit (-, 1, B) with CSN 0

t=5 state
A: (-, 0, A)          
B: (-, 1, B)          
C: (-, 0, A) (-, 1, B)
P: (0, 1, B)          
```

We note that `P` can safely commit `B` because it is the first (and only) tentative update it has from `B`

```
t=6 A-P sync
t=7 P commit (-, 0, A) with CSN 1

t=7 state
A: (-, 0, A)          
B: (-, 1, B)          
C: (-, 0, A) (-, 1, B)
P: (0, 1, B) (1, 0, A)
```

Note `C, P` conflict. A sync between `C-P` would identify `P` as the older replica (its largest CSN is 1, to `C`'s -1). Then `C` would have to rollback its tentative updates.

## Version Vector - Fast Sync

State includes version vectors `VV` now

```
t=30 state
A: (-, 10, X), (-, 20, X), (-, 30, X) (-, 30, Y)
B: (-, 30, Y)

A VV: [A:0, B:0, X:30, Y:30]
B VV: [A:0, B:0, X:0 , Y:30]
```

An `A-B` sync at `t=31` would first exchange `VV`, then `X` would send everything in the range between `B.X` (exclusive) and `A.X` (inclusive), namely `X` updates at `10, 20, 30`. `Y` then Lamport-sorts the tentative updates.

```
t=31 state
A: (-, 10, X), (-, 20, X), (-, 30, X) (-, 30, Y)
B: (-, 10, X), (-, 20, X), (-, 30, X) (-, 30, Y)

A VV: [A:0, B:0, X:30, Y:30]
B VV: [A:0, B:0, X:30, Y:30]
```

## Server Config Changes

Need to trim `VV` of retired servers. Poses a problem for syncing. Consider this scenario. Suppose we start with two servers, `A,P`.

```
t=0 state
A: 
P: 

A VV: [A:0, P:0]
P VV: [A:0, P:0]

t=0 B joins by notifying A
t=1 B updates
t=1 A updates

t=1 state
A: (-, 0, A, B joined) (-, 1, A)
B: (-, 1, B)
P:

A VV: [A:1, B:0, P:0]
B VV: [A:0, B:0, P:0]
P VV: [A:0, P:0]

t=2 A-B sync

t=2 state
A: (-, 0, A, B joined) (-, 1, A) (-, 1, B)
B: (-, 0, A, B joined) (-, 1, A) (-, 1, B)
P:


A VV: [A:1, B:1, P:0]
B VV: [A:1, B:1, P:0]
P VV: [A:0, P:0]
```

We can't keep a retired server in our `VV` to avoid state bloat. Suppose the following plays out:

```
t=3 B decides to retire
t=4 B-P sync

t=4 state
A: (-, 0, A, B joined) (-, 1, A) (-, 1, B)
P: (-, 0, A, B joined) (-, 1, A) (-, 1, B) (-, 3, B, B retiring)

A VV: [A:1, B:1, P:0]
P VV: [A:1, P:0]
```

Now, if we have an `A-P` sync at `t=5`, how do `A,P` tell who's right from version vectors alone? It could be that `P` just has log `(-, 1, A)`, which also has `VV` `[A:1, P:0]`.

Trick: recall ID `B=(0, A)` i.e., pair of time joining and server joined with. 

Thus `P` can figure out locally that `A`'s sent `VV`, `[A:1, (0, A):1, P:0]`, implies `A`'s events about `B` are contained in `P`: `(0, A)` joined is in `P`'s log.

If instead `P` got `[A:1, (1, A):1, P:0]`, this would mean `B` re-joined, and `A` has a longer log.

