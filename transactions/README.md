# Summary 

## Transactions
**Transaction**: a unit of work that may consist of multiple data accesses. It must commit or abort as a **single unit**, and achieve **ACID** properties.

Since consistency is handled by the application, we need to worry about failures (atomicity + durability), and concurrency (isolation).

### Failure Control
**Force Policy**: force all of a transaction's writes to disk before commit  
**Steal Policy**: allow uncommitted writes of transactions to overwrite committed values on disk  
Choose a **no-force** policy (support **redo**ing a committed transaction's writes on disk), and a **steal** policy (support **undo**ing an uncommitted transaction on disk)  

This allows us to implement **Write-Ahead Logging** (WAL):  
* Force all log records **before** any writes to page   
* Transaction is committed when **all** its records are in the log

### Concurrency Control
**Schedule** for transactions is an ordering of the operations performed by those transaction. We want to find two equivalent schedules, i.e. they order all **conflicting** operations in the same way.

**Serial Schedule**: a serial execution of two (or more) transactions.  
![serial schedule image](/transactions/serial_schedule.png)

**Conflict Serializable**: when a schedule is equivalent to **some** serial schedule

But how do we test for serializability? We create a **precedence graph**, where each node ``t`` represents a transaction ``t``, and there is an edge from ``s`` to ``t`` if some action ``s`` precedes and conflicts with some action of ``t``. Then, a schedule is **conflict-serializable** if and only if its precedence graph is **acyclic.** 

![serial schedule image](/transactions/acyclic_graph.png)  
This schedule is conflict-serializable because there is an edge from transfer to sum, but not from sum to transfer.

## Locking Transactions
Now, we need to **ensure** the serializable schedule. This is done with **locks**, which are maintained by the **transaction manager**.  
* Transaction requests lock for an item  
* Transaction manager grants or denies the lock

Essentially, we **allow concurrent** transactions, but add locks/lock manager.
### Lock Types
**Shared**: Need to have before reading object  
**Exclusive**: Need to have before writing to an object  

We can't have a protocol that allows independent grabbing of locks, as non-serializable schedules could still be permitted (strawman).

## 2-Phase Locking
Once a transaction has **released** a lock it is **not allowed to obtain** any other locks. Therefore, there are two phases, a **growing phase** when transactions acquire locks, and a **shrinking phase** when transactions release locks. This ensures serializability!  
![serial schedule image](/transactions/2pl_example.png)  

### Issues:  

* While 2-PL precludes non-serializable schedules, it also precludes some serializable schedules (not an exact one-to-one mapping).
* Deadlock is possible - but can detect deadlock cycles and abort.
* **Phantom Problem**: database is more complex than key-value storage, such that locking individual data items does not ensure serializability

## ARIES
The gold standard in IBM DB2 and Microsoft's SQL Servers. It essentially refined write-ahead logging, repeated history after a crash (via redo), and logged **every** change (even undos during crash recovery).

## OCC
**Optimistic Concurrency Control**: Be optimistic about transactions as we want to have low-overhead for non-conflicting transactions. Thus, we process the transaction as if it would succeed, and only check for serializability at commit time. If it fails, then we abort. 

### Three-Phase Approach  
* *Begin*: Timestamp that marks the beginning of transaction  
* *Modify*: Transaction can read values of committed items, and updates to local copies  
* *Validate*: Validate if it is serializable (otherwise could lead to inconsistent state). See below for more details.
* *Commit*: Apply to database if valid, otherwise restart  

### Validate Phase
During the validation phase the system must ensure that there is *no conflicting concurrency*. 

#### Details
Consider a transaction `X`. For all other transactions `Y`, which are either committed or in validation phase, if all of the following fail, then validation fails and abort `X`.

* `Y` completes commit before `X` starts modify
* `X` starts commit after `Y` completes commit, and ReadSet of `X` and WriteSet of `Y` are disjoint  
* Both ReadSet `X` and WriteSet `X` are disjoint from WriteSet `Y`, and `Y` completes modify phase

## MVCC
**Multi-Version Concurrency Control**   
* Maintain mutliple versions of objects, each with its own timestamp  
* Allocate the correct version to reads  
* Unlike 2PL/OCC, reads never get rejected  
* Essentially all reads execute in one *snapshot*, and all writes execute as one *snapshot*  
* Yields snapshot isolation, which is weaker than serializability  
* Run garbage collection to clean up  
* See black/white marble example if unclear

### Details
Every object `O` has both read and write timestamps.  
**Read Timestamp**: largest timestamp of a transaction that reads `O`  
**Write timestamp**: timestamp of the transaction that wrote `O`  

When executing a transaction `T`:  
1. Find correct version of object `O` by determining the last version written before read snapshot time  
2. Perform write of object `O` or abort if another transaction `T'` exists and has read `O` after `T` (that is if `ReadTS(O) > TS(T)`).

Examples in lecture slides but have many typos.

## Notes
The key to reading the serializability figures.  
![serial schedule image](/transactions/key.png)

While serializability is a guarantee about transactions over one or more objects, linearizability is a guarantee about **single** operations on **single** objects.

2PL is pessimistic (get all locks first), while OCC is optimistic, but then we recheck all read + written items before commit. *We choose OCC in an environment with low data contention, and 2PL when data contention is high.*