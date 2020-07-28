 ### 1.  Requirements clarifications
dsd

 
 ---
### 2. Back-of-the-envelope estimation
- What scale is expected from the system?
- How much storage will we need?
- What network bandwidth usage are we expecting?
 
 ---
 ### 3. System interface definition
Define what APIs are expected from the system. This will not only establish the exact contract expected from the system but will also ensure if we haven’t gotten any requirements wrong
  
 ---
 ### 4. Defining data model
 Defining the data model in the early part of the interview will clarify how data will flow between different components of the system. Later, it will guide for data partitioning and management. The candidate should be able to identify various entities of the system, how they will interact with each other, and different aspects of data management like storage, transportation, encryption, etc.
 
 ---
 ### 5. High-level design
Draw a block diagram with 5-6 boxes representing the core components of our system. We should identify enough components that are needed to solve the actual problem from end-to-end.
 
 ---
 ### 6. Detailed design
 Dig deeper into two or three major components; interviewer’s feedback should always guide us to what parts of the system need further discussion. We should be able to present different approaches, their pros and cons, and explain why we will prefer one approach on the other.
  
 ---
 ### 7. Identifying and resolving bottlenecks
Try to discuss as many bottlenecks as possible and different approaches to mitigate them.

-   Is there any single point of failure in our system? What are we doing to mitigate it?
-   Do we have enough replicas of the data so that if we lose a few servers, we can still serve our users?
-   Similarly, do we have enough copies of different services running such that a few failures will not cause total system shutdown?
-   How are we monitoring the performance of our service? Do we get alerts whenever critical components fail or their performance degrades?
 
 ---
 ### 8. **Summary:**
In short, preparation and being organized during the interview are the keys to be successful in system design interviews. The steps mentioned above should guide you to remain on track and cover all the different aspects while designing a system.	

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE2NjU2NTg1NSwtMTY1MDc2ODIyMV19
-->