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

But how do we test for serializability? We create a **precedence graph**, where each node **t** represents a transaction **t**, and there is an edge from **s** to **t** if some action **s** precedes and conflicts with some action of **t**. Then, a schedule is **conflict-serializable** if and only if its precedence graph is **acyclic.** 

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

## MVCC

Franklin (just summarize + add own folder)

Spanner (just summarize + add own folder)

SPORC (just summarize + add own folder)

## Notes
The key to reading the serializability figures.  
![serial schedule image](/transactions/key.png)

While serializability is a guarantee about transactions over one or more objects, linearizability is a guarantee about **single** operations on **single** objects.
