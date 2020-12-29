# Optimizing Passenger Standalone

These notes summarize [Optimizing Passenger Standalone](https://www.phusionpassenger.com/library/config/standalone/optimization/) from Passenger Docs. 

---

There are two aspects with regard to optimizing Passenger's server performance:

-   The first aspect is **_settings tuning_**. Passenger's default settings are not aimed at optimizing, but at **_safety_**. The defaults are designed to conserve resources, to prevent server overload and to keep web apps up and running. To optimize for performance, you need to tweak some settings whose values depend on your hardware and your environment.    
    Besides Passenger settings, you may also want to tune kernel-level settings.
    
-   The second aspect is **_using performance-enhancing features_**. This requires small application-level changes.

### Minimizing process spawning:
By default, Passenger [spawns and shuts down application processes according to traffic](https://www.phusionpassenger.com/library/config/standalone/dynamic_scaling_vs_fixed_app_processes/). This allows it to use more resources during busy times, while conserving resources during idle times. This is especially useful if you host more than 1 app on a single server: if not all apps are used at the same time, then you don't have to keep all apps running at the same time.

However, spawning a process takes a lot of time (in the order of 10-20 seconds for a Rails app), and CPU usage will be near 100% during spawning. Therefore, while spawning, your server will be slower at performing other activities, such as handling requests.

For consistent performance, it is thus recommended that you configure a [fixed process pool](https://www.phusionpassenger.com/library/config/standalone/dynamic_scaling_vs_fixed_app_processes/?a=fixed): telling Passenger to use a fixed number of processes, instead of spawning and shutting them down dynamically.

Run `passenger start` with `--max-pool-size=N --min-instances=N`, where `N` is the number of processes you want.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5NDk2NDYwOTksMTUwNDM0ODUxN119
-->