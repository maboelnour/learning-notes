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
### The smart spawning method
It is possible to make the different processes share the memory occupied by application and web framework code. This is achieved by utilizing so-called `copy-on-write` semantics of the virtual memory system on modern operating systems. As a side effect, the startup time is also reduced. This technique is exploited by Passenger's `smart` spawn method.

When the `smart` spawn method is being used, Passenger will first create a so-called 'preloader' process. This process loads the entire application along with the web framework, by loading the file `config.ru`. The preloader process doesn't participate in request handling.

Then, whenever Passenger needs a new application process, it will instruct the preloader to create one. The preloader then spawns a child process (with the `fork()` system call). The operating system guarantees that this child process is an exact virtual copy of itself. This child process therefore already has the application code and the web framework code in memory.

Creating a process like this is very fast, about 10 times faster than loading the Ruby application + framework from scratch. On top of that, the OS also applies an optimization called `copy-on-write`. This means that all memory that the child process hasn't modified, is shared with the parent process.

![enter image description here](https://www.phusionpassenger.com/library/indepth/spawn_methods/smart_spawning-45966b9d.png)
### Smart spawning caveats:
Because application processes are created by forking from a preloader process, it will share all file descriptors that are opened by the preloader process. If different application processes write to such a file descriptor at the same time, then their write calls will be interleaved, which may potentially cause problems.

The problem commonly involves socket connections that are unintentionally being shared. You can fix it by closing and reestablishing the connection when Passenger is creating a new application process. So you could insert the following code in your `config.ru`:
```
if defined?(PhusionPassenger)
  PhusionPassenger.on_event(:starting_worker_process) do |forked|
    if forked
      # We're in smart spawning mode.
      ... code to reestablish socket connections here ...
    else
      # We're in direct spawning mode. We don't need to do anything.
    end
  end
end
```
#### Example 1: Memcached connection sharing (harmful)
```
 +--------------------+
 | Preloader          |-----------[Memcached server]
 +--------------------+
```
Suppose that Passenger then proceeds with creating a new application process, which is to process incoming HTTP requests. The result will look like this:
```
 +--------------------+
 | Preloader          |------+----[Memcached server]
 +--------------------+      |
                             |
 +--------------------+      |
 | App process 1      |-----/
 +--------------------+
```
Since a `fork()` makes a (virtual) complete copy of a process, all its file descriptors will be copied as well. What we see here is that Preloader and App process 1 both share the same connection to Memcached.

Now supposed that your site gets a sudden large surge of traffic, and Passenger decides to spawn another process. It does so by forking Preloader. The result is now as follows:
```
 +--------------------+
 | Preloader          |------+----[Memcached server]
 +--------------------+      |
                             |
 +--------------------+      |
 | App process 1      |-----/|
 +--------------------+      |
                             |
 +--------------------+      |
 | App process 2      |-----/
 +--------------------+
```
As you can see, App process 1 and App process 2 have the same Memcached connection.

Suppose that users Joe and Jane visit your website at the same time. Joe's request is handled by App process 1, and Jane's request is handled by App process 2. Both application processes want to fetch something from Memcached. Suppose that in order to do that, both handlers need to send a `FETCH` command to Memcached.

But suppose that, after App process 1 having only sent `FE`, a context switch occurs, and App process 2 starts sending a `FETCH` command to Memcached as well. If App process 2 succeeds in sending only one byte, `F`, then Memcached will receive a command which begins with `FEF`, a command that it does not recognize. In other words: the data from both handlers get interleaved. And thus Memcached is forced to handle this as an error.

This problem can be solved by reestablishing the connection to Memcached after forking:
```
 +--------------------+
 | Preloader          |------+----[Memcached server]
 +--------------------+      |                   |
                             |                   |
 +--------------------+      |                   |
 | App process 1      |-----/|                   |
 +--------------------+      |                   |  <--- created this
                             X                   |       new
                                                 |       connection
                             X <-- closed this   |
 +--------------------+      |     old           |
 | App process 2      |-----/      connection    |
 +--------------------+                          |
           |                                     |
           +-------------------------------------+
```
App process 2 now has its own, separate communication channel with Memcached. The code in `config.ru` looks like this:

```
if defined?(PhusionPassenger)
  PhusionPassenger.on_event(:starting_worker_process) do |forked|
    if forked
      # We're in smart spawning mode.
      reestablish_connection_to_memcached
    else
      # We're in direct spawning mode. We don't need to do anything.
    end
  end
end
```

---

### Am I responsible for reestablishing database connections after the preloader has forked a child process?

It depends. Passenger automatically reestablishes the ActiveRecord primary database connection. The vast majority of Rails only uses ActiveRecord (not any other database libraries), and only with a single database connection.

If your application uses any database libraries besides ActiveRecord, OR you use ActiveRecord with more than one database connection, then you are responsible for reestablishing all connections after forking. Use the [smart spawning hooks](https://www.phusionpassenger.com/library/indepth/ruby/spawn_methods/#smart-spawning-hooks).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4ODcwMTEzNjAsLTIxMDY5NDIzNzMsMT
MxNTgxNzMwMSwtMTg5MjI2MjQ3LC01MTY5NzE5MTJdfQ==
-->