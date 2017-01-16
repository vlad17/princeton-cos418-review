# Summary

## Conflict Resolution
Concurrent writes can conflict and is encountered in many different settings (Bayou, Dynamo, COPS).

The general approach to conflict resolution is to encode operations as incremental updates. For instance, if Alice has $50, and Bob has $20, we do Alice -$10, and Bob +$10, instead of Alice = $40, and Bob = $30.

A key observation: this only works because operations are **commutative** and **invertible**! It *does not* work otherwise.

## Operational Transform
General idea is to take advantage of commutative property of operation. Say that we have two servers, `A` and `B`, both in state `X` with two respective operations `a` and `b`. Even if both servers apply their own operations first, we should still end at the same final state. 

 
An **operational transform** transforms an op `a` into `a'`, such that `X` * `a` * `b'` == `X` * `b` * `a'`. Therefore, on `A` we want to transform `a` across all operations applied since `X` (likewise on `B`).  

*Server A*  
1. `A` applies `a`: State `X` * `a`  
2. `A` transforms `b`: `b'` = OT(`b`, {`a`})  
3. `A` applies `b'`: State `X` * `a` * `b'`  
  
*Server B*  
1. `B` applies `b`: State `X` * `b`  
2. `B` transforms `a`: `a'` = OT(`a`, {`b`})  
3. `B` applies `a'`: State `X` * `b` * `a'`  