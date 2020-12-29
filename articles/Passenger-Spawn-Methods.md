# Spawn methods explained  for Ruby developers
These notes summarize [Spawn methods explained](https://www.phusionpassenger.com/library/indepth/ruby/spawn_methods/) from Passenger Docs. 

---

At its core, Passenger is an HTTP proxy and process manager. It spawns application processes and forwards incoming HTTP request to one of them.

Passenger supports so-called "smart spawning" (currently for Ruby applications only), which leverages virtual memory copy-on-write semantics of the operating system in order to reduce overall memory usage.

---

### The most straightforward and traditional way: direct spawning

Passenger could create a new Ruby process, which will then load the web framework (e.g. Rails) along with the application code. This process then enters a request handling main loop.

This is the most straightforward way to spawn processes, and each process contains a full copy of the application code and the web framework in memory.

While this works well, it's not as efficient as it could be because each process has its own **private** copy of the application code as well as the web framework. This wastes memory as well as startup time.

![enter image description here](https://www.phusionpassenger.com/library/indepth/spawn_methods/direct_spawning-7fd82545.png)
## [](https://www.phusionpassenger.com/library/indepth/ruby/spawn_methods/#the-smart-spawning-method)The smart spawning method
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA3NTU3Mjk0MCwtNTE2OTcxOTEyXX0=
-->