# Summary
VMware's vSphere is a system that provides fault-tolerant virtual machines. This system replicates the execution of a primary virtual
machine (VM) via a backup virtual machine on another server. It is designed for correctness and consistency of the replicated VM outputs despite failures (system behaves like a single server). It generally delivers high performance and low logging bandwidth overhead.

## Goals
* Replication of the whole virtual machine
* Completely transparent to applications and clients
* High availability for any existing software

## Setup
Two virtual machines (a primary and backup) on different bare metal, with a shared disk and logging channel over a shared network.

## I/O
VM inputs include:  

* Incoming network packets
* Disk Reads
* Keyboard and mouse events
* Clock timer interrupt events

VM outputs include: 

* Outgoing network packets
* Disk writes

## Algorithm
Primary sends inputs to the backup, while the backup drops its outputs. The primary-backup relationship is sustained with heartbeats; if the primary fails, the backup takes over.

### Log-Based VM Replication
The hypervisor (virtual machine monitor) at the primary logs any causes of non-determinism, such as input events and non-deterministic instructions. It then sends log entries to backup hypervisor over the logging channel. The backup hypervisor replays the log entries and delivers the same input as the primary

### Primary to Backup Failover
When the backup VM takes over, its execution **must** be consistent with outputs that the primary VM has already sent. However, there is no way that the backup can determine if a primary crashed immediately before or after sending its last output, so this packet may be duplicated (that's what TCP is for).

**FT Protocol**: the primary logs each output operation, and **delays** any output until the backup acknowledges it.

### Avoiding Split Brain
The logging channel may break, and we want to avoid two primaries. The primary and backup each run UDP hearbeats, monitoring loggnig traffic from their peer. Before a backup becomes the primary, or a primary finds a new backup, they execute an **atomic test-and-set** on a variable in shared storage. If the replica finds the variable is already set, it aborts. 