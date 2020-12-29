# Optimizing Passenger Standalone

These notes summarize [Optimizing Passenger Standalone](https://www.phusionpassenger.com/library/config/standalone/optimization/) from Passenger Docs. 

---

There are two aspects with regard to optimizing Passenger's server performance:

-   The first aspect is **_settings tuning_**. Passenger's default settings are not aimed at optimizing, but at **_safety_**. The defaults are designed to conserve resources, to prevent server overload and to keep web apps up and running. To optimize for performance, you need to tweak some settings whose values depend on your hardware and your environment.    
    Besides Passenger settings, you may also want to tune kernel-level settings.
    
-   The second aspect is **_using performance-enhancing features_**. This requires small application-level changes.

### Minimizing process spawning:
By default, Passenger [spawns and shuts down application processes according to traffic](https://www.phusionpassenger.com/library/config/standalone/dynamic_scaling_vs_fixed_app_processes/). 
However, spawning a process takes a lot of time (in the order of 10-20 seconds for a Rails app), and CPU usage will be near 100% during spawning. Therefore, while spawning, your server will be slower at performing other activities, such as handling requests.

For consistent performance, it is thus recommended that you configure a [fixed process pool](https://www.phusionpassenger.com/library/config/standalone/dynamic_scaling_vs_fixed_app_processes/?a=fixed): telling Passenger to use a fixed number of processes, instead of spawning and shutting them down dynamically.

Run `passenger start` with `--max-pool-size=N --min-instances=N`, where `N` is the number of processes you want.


### Maximizing throughput

The amount of throughput that Passenger handles is proportional to the number of processes or threads that you've configured. More processes/threads generally means more throughput, but there is an upper limit. If you increase the number of processes/threads even further, then performance may even go down.

The optimal value depends on the hardware and the environment. This section will provide you with formulas to calculate that optimal value. The following factors are involved in calculation:

-   **Memory**. More processes implies a higher memory usage. If too much memory is used then the machine will hit swap, which slows everything down. Threads use less memory, so prefer threads when possible. You can create tens of threads in place of one process.
-   **Number of CPUs**. True (hardware) concurrency cannot be higher than the number of CPUs. In theory, if all processes/threads on your system use the CPUs constantly, then:
    
    -   You can increase throughput up to `NUMBER_OF_CPUS` processes/threads.
    -   Increasing the number of processes/threads after that point will increase virtual (software) concurrency, but will not increase true (hardware) concurrency and will not increase maximum throughput.
    
    Having more processes than CPUs may decrease total throughput a little thanks to context switching overhead, but the difference is not big because OSes are good at context switching these days.
    
    On the other hand, if your CPUs are not used constantly, e.g. because they’re often blocked on I/O, *then the above does not apply and increasing the number of processes/threads does **increase concurrency and throughput**, at least until the CPUs are saturated*.
    
-   **Blocking I/O**. This covers all blocking I/O, including *hard disk access* *latencies*, *database call latencies*, *web API calls*, etc. Handling input from the client and output to the client does not count as blocking I/O, because Passenger has buffering layers that relief the application from worrying about this.
    
    The more blocking I/O calls your application process/thread makes, the more time it spends on waiting for external components. While it’s waiting it does not use the CPU, so that’s when another process/thread should get the chance to use the CPU. If no other process/thread needs CPU right now (e.g. all processes/threads are waiting for I/O) then CPU time is *essentially wasted*. Increasing the number processes or threads decreases the chance of CPU time being wasted. It also increases concurrency, so that clients do not have to wait for a previous I/O call to be completed before being served.
    
The formulas in this section assume that your machine is dedicated to Passenger. If your machine also hosts other software (e.g. a database) then you'll need to tweak the formulas a little bit.

### Tuning the application process and thread count:

##### Step 1: determining the application's memory usage:
You should first figure out how much memory your application typically needs. Every application has different memory usage patterns, so the typical memory usage is best determined by observation.

Run your app for a while, then run `passenger-status` at different points in time to examine memory usage. 

##### Step 2: determine the system's limits:
First, let's define the maximum number of (single-threaded) processes, or the total number of threads, that you can comfortably have given the amount of RAM you have. This is a reasonable upper limit that you can reach without degrading system performance. This number is not the final optimal number, but is merely used for further caculations in later steps.

There are two formulas that we can use, depending on what kind of concurrency model your application is using in production.

*Purely single-threaded multi-process formula*

If you didn't explicitly configure multithreading, then you are using using this concurrency model.
The formula is then as follows:
```
max_app_processes = (TOTAL_RAM * 0.75) / RAM_PER_PROCESS
```

It is derived as follows:

-   `(TOTAL_RAM * 0.75)`: We can assume that there must be at least 25% of free RAM that the operating system can use for other things. The result of this calculation is the RAM that is freely available for applications. If your system runs a lot of services and thus has less memory available for Passenger and its apps, then you should lower `0.75` to some constant that you think is appropriate.
-   `/ RAM_PER_PROCESS`: Each process consumes a roughly constant amount of RAM, so the maximum number of processes is a single devision between the aforementioned calculation and this constant.




<!--stackedit_data:
eyJoaXN0b3J5IjpbODA3MTY1MzQyLDE1MDQzNDg1MTddfQ==
-->