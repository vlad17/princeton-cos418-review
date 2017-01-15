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

## Locking Transactions

## OCC

## MVCC

Franklin (just summarize + add own folder)

Spanner (just summarize + add own folder)

SPORC (just summarize + add own folder)
