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

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzMDE0MTUzMzJdfQ==
-->