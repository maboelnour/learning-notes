reasons for distributed computing

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


### Map Reduce pattern



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NjQwMTU1ODAsMzAwMjAxNDIsLTIzNz
IzMjgwNSwxMDk3OTczNTkxLC0xNjQ1NzkzMjAxLC0xOTA3Njc1
NDMwLC0xODQxNzQzMzMxXX0=
-->