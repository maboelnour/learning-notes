# PostgreSQL Indexes

## Introduction

Once an index is created, no further intervention is required: the system will update the index when the table is modified, and it will use the index in queries when **it thinks doing so would be more efficient than a sequential table scan. But you might have to run the ANALYZE command regularly to update statistics to allow the query planner to make educated decisions.** See [Chapter 14](https://www.postgresql.org/docs/12/performance-tips.html) for information about how to find out whether an index is used and when and why the planner might choose not to use an index.

 Indexes can also benefit UPDATE and DELETE commands with search conditions. Indexes can moreover be used in join searches. Thus, an index defined on a column that is part of a join condition can also significantly speed up queries with joins.
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzkzMDU1ODM2XX0=
-->