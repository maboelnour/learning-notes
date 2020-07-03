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

 1. **Sticky sessions with Stateful nodes**
- Some web applications use sticky sessions, which assign each user to a specific web server node when they first visit. Once assigned, that node satisfies all of that user’s page re quests for the duration of the visit. This is supported in two places: the load balancer ensures that each user is directed to their assigned node, while the web server nodes store session state for users between page requests.
When stateful nodes hold the only copy of a user’s session state, there are user experience challenges. If the node that is managing the sticky session state for a user goes away, that user’s session state goes with it. This may force a user to log in again or cause the contents of a shopping cart to vanish.
- Sessions may also be unevenly distributed as node instances come and go. Suppose your web tier has two web server nodes, each with 1,000 active sessions. You add a third node to handle the expected spike in traffic during lunchtime. The typical load balancer ran­domly distributes new requests across all nodes. It will not have enough information to send new sessions to the newly added node until it also has 1,000 active sessions. It is effectively “catching up” to the other nodes in the rotation. Each of the 3 nodes will get approximately one-third of the next 1,000 new sessions, resulting in an imbalance.

___
[**Enabling Session Persistence with NGINX**](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/#enabling-session-persistence)
Session persistence means that NGINX Plus identifies user sessions and routes all requests in a given session to the same upstream server.

- **IP Hash** The server to which a request is sent is determined from the client IP address. In this case, either the first three octets of the IPv4 address or the whole IPv6 address are used to calculate the hash value. The method guarantees that requests from the same address get to the same server unless it is not available.
	```
	upstream backend {
	    ip_hash;
	    server backend1.example.com;
	    server backend2.example.com;
	}
	```

- **Sticky cookie** NGINX Plus adds a **session cookie** to the first response from the upstream group and identifies the server that sent the response. The client’s next request contains the cookie value and NGINX Plus route the request to the upstream server that responded to the first request: 
    ```
    upstream backend {
	    server backend1.example.com;
	    server backend2.example.com;
	    sticky cookie srv_id expires=1h domain=.example.com path=/;
   }
    ```
- **Sticky route** NGINX Plus assigns a **“route”** to the client when it receives the first request. All subsequent requests are compared to the [`route`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#route) parameter of the `server` directive to identify the server to which the request is proxied. The route information is taken from either a cookie or the request URI.
    ```
    upstream backend {
        server backend1.example.com route=a;
        server backend2.example.com route=b;
        sticky route $route_cookie $route_uri;
    }
    ```
- **Sticky learn** NGINX Plus first finds **session identifiers** by inspecting requests and responses. Then NGINX Plus **“learns”** which upstream server corresponds to which session identifier. Generally, these identifiers are passed in a HTTP cookie. If a request contains a session identifier already “learned”, NGINX Plus forwards the request to the corresponding server:
	```
	upstream backend {
	   server backend1.example.com;
	   server backend2.example.com;
	   sticky learn
	       create=$upstream_cookie_examplecookie
	       lookup=$cookie_examplecookie
	       zone=client_sessions:1m
	       timeout=1h;
	}
	```
___
2. **Session state without stateful nodes**

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNTU3MDM1MiwtMjMxMzMwMzk1LC0xNT
YyMzQ3NTI5LC0xMTM5NDYyNDYyLC0xMzAxNDE1MzMyXX0=
-->