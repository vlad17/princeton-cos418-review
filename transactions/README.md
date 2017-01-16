# Summary 

## Transactions
**Transaction**: a unit of work that may consist of multiple data accesses. It must commit or abort as a **single unit**, and achieve **ACID** properties.

Since consistency is handled by the application, we need to worry about failures (atomicity + durability), and concurrency (isolation).

### Failure Control

**Force Policy**: force all of a transaction's writes to disk before commit [note this refers to the transaction's changes themselves; the transaction itself may still be forced to disk before commit for durability guarantees] 
**Steal Policy**: allow uncommitted writes of transactions to overwrite committed values on disk 

Force policy: `fsync` on every transaction's critical path, slow. No-force: might have to re-do transactions that are commited but not yet forced.
 
No-Steal policy: transaction scheduling inflexible. **Steal**: might have to undo transactions that were written, then aborted.

Assume we can support redo, undo for no-force, steal. This allows us to implement **Write-Ahead Logging** (WAL):  

* Force all log records **before** any writes to page   
* Transaction is committed when **all** its records are in the log

When _coupled with proper crash recovery_, force/no-steal/WAL gives fast **D**urability in **ACID**.

### Concurrency Control

**Schedule** for transactions is an ordering of the operations performed by those transaction. We want to find two equivalent schedules, i.e. they order all **conflicting** operations in the same way.

**Serial Schedule**: a serial execution of two (or more) transactions.  
![serial schedule image](/transactions/serial_schedule.png)

**Conflict Serializable**: when a schedule is equivalent to **some** serial schedule

Test for serializability: create a **precedence graph**, where each node ``t`` represents a transaction ``t``, and there is an edge from ``s`` to ``t`` if some action ``s`` precedes and conflicts with some action of ``t``. Then, a schedule is **conflict-serializable** if and only if its precedence graph is **acyclic.** 

## Locking Transactions
Now, we need to **ensure** the serializable schedule. This is done with **locks**, which are maintained by the **transaction manager**.  
* Transaction requests lock for an item  
* Transaction manager grants or denies the lock

Essentially, we **allow concurrent** transactions, but add locks/lock manager.
### Lock Types
**Shared**: Need to have before reading object  
**Exclusive**: Need to have before writing to an object  

## 2-Phase Locking
Once a transaction has **released** a lock it is **not allowed to obtain** any other locks. Therefore, there are two phases, a **growing phase** when transactions acquire locks, and a **shrinking phase** when transactions release locks. This ensures serializability!  
![serial schedule image](/transactions/2pl_example.png)  

### Issues:  

* While 2-PL precludes non-serializable schedules, it also precludes some serializable schedules (not an exact one-to-one mapping).
* Deadlock is possible - but can detect deadlock cycles and abort (or put ordering on cycles).
* **Phantom Problem**: database multi-row ops more complex than key-value storage: locking individual data items does not ensure serializability

## Serializability vs Linearizability

**Linearizability** is equal to strong consistency: briefly, everyone agrees on a history of operations, where writes occur instantaneously at some real point in time - single object equivalent (i.e., the replicated state in a replicated state machine).

**Serializability** is a guarantee about groups of transactions: the concurrent application of multiple transactions is equivalent to _some_ serial transaction in some order ([more notes](http://www.bailis.org/blog/linearizability-versus-serializability/)).

## Crash Recovery

Introduced in **ARIES**: the gold standard in IBM DB2 and Microsoft's SQL Servers. It essentially refined write-ahead logging, repeated history after a crash (via redo), and logged **every** change (even undos during crash recovery).

`redo + undo + WAL + checkpointing + crash handling = ACID`

## OCC
**Optimistic Concurrency Control**: Be optimistic about transactions as we want to have low-overhead for non-conflicting transactions. Thus, we process the transaction as if it would succeed, and only check for serializability at commit time. If it fails, then we abort. 

### Three-Phase Approach  
* *Begin*: Timestamp that marks the beginning of transaction  
* *Modify*: Transaction can read values of committed items, and updates to local copies  
* *Validate*: Validate if it is serializable (otherwise could lead to inconsistent state). See below for more details.
* *Commit*: Apply to database if valid, otherwise restart  

### Validate Phase
During the validation phase the system must ensure that there is *no conflicting concurrency*. 

### Comparison to 2PL

2PL is pessimistic (get all locks first), while OCC is optimistic, but then we recheck all read + written items before commit. *We choose OCC in an environment with low data contention, and 2PL when data contention is high.*

## MVCC
**Multi-Version Concurrency Control**   
* Maintain mutliple versions of objects, each with its own timestamp  
* Allocate the correct version to reads  
* Unlike 2PL/OCC, reads never get rejected  
* Essentially all reads execute in one *snapshot*, and all writes execute as one *snapshot*  
* Yields snapshot isolation, which is weaker than serializability  
* Run garbage collection to clean up  

### Details
Every object `O` has both read and write timestamps.  
**Read Timestamp**: largest timestamp of a transaction that reads `O`  
**Write timestamp**: timestamp of the transaction that wrote `O`  

When executing a transaction `T`:  
1. Find correct version of object `O` by determining the last version written before read snapshot time  
2. Perform write of object `O` or abort if another transaction `T'` exists and has read `O` after `T` (that is if `ReadTS(O) > TS(T)`).

Examples in lecture slides but have many typos.

# Detailed Explanation

## OCC Validation

Consider a transaction `X`. For all other transactions that whose modify period intersected with `X`'s, one of the following must hold. Let the other transaction be `Y`.

* `Y` completes commit before `X` starts modify
* `X` starts commit after `Y` completes commit, and ReadSet of `X` and WriteSet of `Y` are disjoint  
* Both ReadSet `X` and WriteSet `X` are disjoint from WriteSet `Y`, and `Y` completes modify phase

If all of the above fail for any single transaction `Y`, then validation fails and abort `X`.

We can use notions of "before" and "after" by assuming we have some access to "real time" or an atomic counter.

## Useful Equations

Linearizability where operations are transactions _implies_ Serializability.

For key-value storage: Strict Serializability = Linearizability (operations are transactions) = Serializability (on transactions) + Linearizability (operations are reads/writes)

Linearizability = **C**onsistency in **CAP**

Serializability = **I**solation

Both OCC and 2PL offer both **I** of **ACID**.

Crash recovery (as in ARIES) + WAL offers **D** (checkpointing makes recovery faster). Alternatively, we can have **D** from force or no-steal systems.

**A** is ensured by transaction commit/abort semantics.

**C** in **ACID** is ensured in part by OCC and 2PL (they make sure at most one person is writing to a location at one time) and by the database code. The other part comes from the database code never writing invalid values into the data (through careful `fsync`-ing (no partial writes) and type safety.

## Checking For a Serializable Schedule

The key to reading the serializability figures.  
![serial schedule image](/transactions/key.png)

![serial schedule image](/transactions/acyclic_graph.png)  
This schedule is conflict-serializable because there is an edge from transfer to sum, but not from sum to transfer. The following is not.

![serial schedule image](/transactions/cyclic.png)  

## 2PL Permits Interleaved Access

TODO: Summarize (preferably with ASCII art) L15 slides 37

## 2PL Doesn't Exploit Full Concurrency

TODO: Summarize (preferably with ASCII art) L15 slides 38

## Undisciplined Locking causes Non-serializable Schedules

TODO: Give example (preferably with ASCII art) of the above, and how it would be detected.

## ARIES Data Structures

TODO: Summarize L15 slides 43-46

## ARIES Crash Recovery Trace

TODO: Summarize (preferably with ASCII art) L15 slides 47-50

## Black/White Marble Example

TODO: Snapshot isolation but not Serializable example (preferably with ASCII art) L16 slide 14
