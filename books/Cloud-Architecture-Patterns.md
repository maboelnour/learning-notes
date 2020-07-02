# Cloud Architecture Patterns

![Cloud Architecture Patterns](https://images-na.ssl-images-amazon.com/images/I/413c-GZAMZL._SX372_BO1,204,203,200_.jpg)
Summarize the `Cloud Architecture Patterns` with external resources.

## Chapter 1: Scalability Primer

- To vertically scale up is to increase overall application capacity by increasing the
resources within existing nodes. Scaling up is limited by the utilizable capability of available hardware.
- To horizontally scale out is to increase overall application capacity by adding nodes.
Horizontal scaling is more efficient with homogeneous nodes.

These scaling approaches are neither **mutually exclusive nor all-or-nothing**. Any application is capable of vertically scaling up, horizontally scaling out, neither, or both. For
example, parts of an application might only vertically scale up, while other parts might
also horizontally scale out.

Vertical scaling is often hardware- and infrastructure-focused—we “**throw hardware at the problem**”—whereas horizontal scaling is development- and architecture-focused. Depending on which scal­ing strategy is employed, the responsibility may fall to specialists in different depart­ments, complicating matters for some companies.


#### Describing Scalability:
Scalability can be described by: 
- Concurrent users: the number of users with activity within a specific time interval
(such as ten minutes).
- Response time: the elapsed time between a user initiating a request (such as by
clicking a button) and receiving the round-trip response.

`Cloud-native applications allocate resources horizontally, and scalability is only one benefit.`

## Chapter 2: Horizontally Scaling Compute Pattern 

The Horizontal Scaling Compute Pattern effectively deals with the following challenges:
- Cost-efficient scaling of compute nodes is required, such as in the web tier or service
tier.
- Application capacity requirements exceed (or may exceed after growth) the capacity
of the largest available compute node.
- Application capacity requirements vary seasonally, monthly, weekly, or daily, or are
subject to unpredictable spikes in usage.
- Application compute nodes require minimal downtime, including resilience in the
event of hardware failure, system upgrades, and resource changes due to scaling.

This pattern is typically used in combination with the **Node Termination Pattern** and the **Auto-Scaling Pattern**.

**Cloud Scaling is Reversible** Cloud scaling is easily reversed. Costs vary in proportion to scale as scale varies over time.

#### Managing Session State:

 1. **Sticky sessions with Stateful servers**

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM4Njc4NDg4NSwtMTMwMTQxNTMzMl19
-->