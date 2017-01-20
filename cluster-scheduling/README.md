# Summary

CPU/Memory/Bandwidth allocation problem.

Assumptions: can subdivide resources evenly.

## Single-Resource

Users get shares according to their importance, but if a user wants less then their excess should be used: **max-min fairness**.

**Weighted fair queueing** as a solution to the allocation problem: find an `f` such that over every user `i`, `sum_i min(request_i, f * weight_i)` is equal to the capacity.

Theoretical properties: everyone gets at least their weighted share under complete contention. Will get less if less demanded. No strategy helps users.

## Multi-Resource

**Asset-fairness** (sum over all resources of user's share) not optimal: can build cases where splitting cluster up according to weight works better. Can also be gamed.

**Dominant resource fairness** - user's dominant resource share is the share of the resource they own most of amongst all their partially owned resources. Solution is to apply max-min fairness to dominant resource shares.

Online DRF scheduler: offer next task to user with smallest dominant share.

## Existing Solutions

A framework (like Hadoop or spark) has many jobs, which include multiple stages and dependencies between sub-tasks. Jobs don't have intra-job dependencies.

Global scheduler (e.g., Google **Borg**): accepts all jobs online, schedules everything at lowest level. Complex to implement, though can be optimal. **Borg** has a centralized Borgmaster node for scheduling + local borglets for monitoring jobs.

Resource offers (e.g., **Mesos**): create resource offers to frameworks (requires application interaction). **Ramp-up time** (time until job demands are met) can be improved with **preemption** of tasks or **migration** (can move tasks)

Example of online DRF run:

![online drf](/cluster-scheduling/online-drf.png)

