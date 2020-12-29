# Optimizing Passenger Standalone

These notes summarize [Optimizing Passenger Standalone](https://www.phusionpassenger.com/library/config/standalone/optimization/) from Passenger Docs. 

---

There are two aspects with regard to optimizing Passenger's server performance:

-   The first aspect is **_settings tuning_**. Passenger's default settings are not aimed at optimizing, but at **_safety_**. The defaults are designed to conserve resources, to prevent server overload and to keep web apps up and running. To optimize for performance, you need to tweak some settings whose values depend on your hardware and your environment.    
    Besides Passenger settings, you may also want to tune kernel-level settings.
    
-   The second aspect is **_using performance-enhancing features_**. This requires small application-level changes.

### Minimizing process spawning:

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTUwNDM0ODUxN119
-->