# Summary

Raft implements distributed consensus in a more modular way than Paxos by using a strong leader and separating the algorithm into four modules that are not fully independent but intersect minimally: **leader election**, **log replication**, **snapshotting**, and **membership changes**. The last two were not studied in this course.


