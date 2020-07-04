reasons fofr distributed computing

- parallelism
- fault tolerance
- physical
- security / isolation

hard of DS is a lot of unexpected failure patterns.

challenges:
- concurrency
- partial failure
- performance 

DS infrastructure (Abstractions):
- Storage
- Communication
- Computation.

Implementation:
- RPC
- Threads
- Concurrency control

Performance:
- Scalability  

Fault Tolerance: 
- Availability: under a predefined set of failures, the service should u[ and running.
- Recoverability.


Consistency:
- `put(k,v)` then `get(k)=v` a cross all replicas.
- Strong Consistency requires a lot of strong communication.
- Lowering communication cost by putting the replicas close to each other collide with `fault tolerant` system.




<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY4NjEyNjc5MywtMjM3MjMyODA1LDEwOT
c5NzM1OTEsLTE2NDU3OTMyMDEsLTE5MDc2NzU0MzAsLTE4NDE3
NDMzMzFdfQ==
-->