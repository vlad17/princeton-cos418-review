# Summary

NFS is a specification for remote (network) file systems.

Providing an abstraction equivalent to POSIX file systems with multiple processes running is the (unattainable) goal; informally, this is **external consistency**. It would require a synchronous RPC call with every modification.

NFSv2

* Read-ahead
* Write-through
* File handles store all per-client state (inode number, generation) -> server is stateless in that it only needs to manage per-file state (i.e., multiple generations).

NFSv3

* Write-behind (wait until `close/fsync` to issue changes). Coarsifies consistency to session-level.
* Read cache consistency verified by client maintaining `last_validation` time. If `current_time - last_validation < threshold`, clocks are consistent, and writes wait longer than `threshold` to take effect, can read from cache.
* Challenge: session-concurrent writers change each other's offsets. Resolution: stateful server invalidates previous writer upon changes by new one.

NFSv4

* Use leases (with clock-skew padding, they are basically time-bounded locks given to clients) to avoid client failure to unlock.
* Write leases are exclusive, read leases can overlap.


