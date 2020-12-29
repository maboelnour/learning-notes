# Spawn methods explained  for Ruby developers
These notes summarize [Spawn methods explained](https://www.phusionpassenger.com/library/indepth/ruby/spawn_methods/) from Passenger Docs. 

---

At its core, Passenger is an HTTP proxy and process manager. It spawns application processes and forwards incoming HTTP request to one of them.

Passenger supports so-called "smart spawning" (currently for Ruby applications only), which leverages virtual memory copy-on-write semantics of the operating system in order to reduce overall memory usage.


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwMDczMzkwODEsLTUxNjk3MTkxMl19
-->