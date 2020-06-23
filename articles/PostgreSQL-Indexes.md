# PostgreSQL Indexes

## Introduction:

Once an index is created, no further intervention is required: the system will update the index when the table is modified, and it will use the index in queries when **it thinks doing so would be more efficient than a sequential table scan. But you might have to run the ANALYZE command regularly to update statistics to allow the query planner to make educated decisions.** See [Chapter 14](https://www.postgresql.org/docs/12/performance-tips.html) for information about how to find out whether an index is used and when and why the planner might choose not to use an index.

 Indexes can also benefit `UPDATE` and `DELETE` commands with search conditions. Indexes can moreover be used in join searches. Thus, an index defined on a column that is part of a join condition can also significantly speed up queries with joins.
 
Creating an index on a large table can take a long time. By default, PostgreSQL allows reads (`SELECT` statements) to occur on the table in parallel with index creation, but writes (`INSERT`, `UPDATE`, `DELETE`) are blocked until the index build is finished. In production environments this is often unacceptable. It is possible to allow writes to occur in parallel with index creation, but there are several caveats to be aware of — for more information see [Building Indexes Concurrently](https://www.postgresql.org/docs/12/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY "Building Indexes Concurrently").

After an index is created, the system has to keep it synchronized with the table. This adds overhead to data manipulation operations. Therefore indexes that are seldom or never used in queries should be removed.

## Index Types:

 1. B-trees indexes
The optimizer can also use a B-tree index for queries involving the pattern matching operators `LIKE` and `~` if the pattern is a constant and is anchored to the beginning of the string — for example, `col LIKE 'foo%' `or `col ~ '^foo'`, but not `col LIKE '%bar'`.
[\[B-Tree index structure in InnoDB\]](https://blog.jcole.us/2013/01/10/btree-index-structures-in-innodb/)

2. Hash indexes
Hash indexes can only handle simple equality comparisons. The query planner will consider using a hash index whenever an indexed column is involved in a comparison using the = operator.

3. GiST indexes
GiST indexes are not a single kind of index, but rather an infrastructure within which many different indexing strategies can be implemented. Accordingly, the particular operators with which a GiST index can be used vary depending on the indexing strategy (the operator class)

4. SP-GiST indexes
SP-GiST indexes, like GiST indexes, offer an infrastructure that supports various kinds of searches. SP-GiST permits implementation of a wide range of different non-balanced disk-based data structures, such as quadtrees, k-d trees, and radix trees (tries).

5. GIN indexes
GIN indexes are `inverted indexes` [\[inverted indexes vs forward indexes\]](https://github.com/bhatman17/inverted-index-series/tree/master/introduction/diff-frwd-invrrtd-index) which are appropriate for data values that contain multiple component values, such as arrays. An inverted index contains a separate entry for each component value, and can efficiently handle queries that test for the presence of specific component values.

  
 6. BRIN indexes
BRIN indexes (a shorthand for Block Range INdexes) store summaries about the values stored in consecutive physical block ranges of a table. Like GiST, SP-GiST and GIN, BRIN can support many different indexing strategies, and the particular operators with which a BRIN index can be used vary depending on the indexing strategy. For data types that have a linear sort order, the indexed data corresponds to the minimum and maximum values of the values in the column for each block range.

## Multicolumn Indexes

A multicolumn B-tree index can be used with query conditions that involve any subset of the index's columns, but the index is most efficient when there are constraints on the leading (leftmost) columns. The exact rule is that equality constraints on leading columns, plus any inequality constraints on the first column that does not have an equality constraint, will be used to limit the portion of the index that is scanned. Constraints on columns to the right of these columns are checked in the index, so they save visits to the table proper, but they do not reduce the portion of the index that has to be scanned.

For example, given an index on (a, b, c) and a query condition `WHERE a = 5 AND b >= 42 AND c < 77`, the index would have to be scanned from the first entry with `a = 5` and `b = 42` up through the last entry with `a = 5`. Index entries with `c >= 77` would be skipped, but they'd still have to be scanned through [[full index scan vs full table scan]](https://dzone.com/articles/full-table-scan-vs-full-index).

This index could in principle be used for queries that have constraints on `b` and/or `c` with no constraint on `a` — but the entire index would have to be scanned, **so in most cases the planner would prefer a sequential table scan over using the index.**

Multicolumn indexes should be used sparingly. In most situations, an index on a single column is sufficient and saves space and time. Indexes with more than three columns are unlikely to be helpful unless the usage of the table is extremely stylized

## Combining Multiple Indexes

A single index scan can only use query clauses that use the index's columns with operators of its operator class and are joined with AND. For example, given an `index on (a, b)` a query condition like `WHERE a = 5 AND b = 6` could use the index, but a query like `WHERE a = 5 OR b = 6` could not directly use the index.

Fortunately, PostgreSQL has the ability to combine multiple indexes (including multiple uses of the same index) to handle cases that cannot be implemented by single index scans. The system can form AND and OR conditions across several index scans. For example, a query like `WHERE x = 42 OR x = 47 OR x = 53 OR x = 99` could be broken down into four separate scans of an index on x, each scan using one of the query clauses. The results of these scans are then ORed together to produce the result. Another example is that if we have separate indexes on x and y, one possible implementation of a query like `WHERE x = 5 AND y = 6` is to use each index with the appropriate query clause and then AND together the index results to identify the result rows.

To combine multiple indexes, the system scans each needed index and prepares a `bitmap` in memory giving the locations of table rows that are reported as matching that index's conditions. The bitmaps are then ANDed and ORed together as needed by the query. Finally, the actual table rows are visited and returned. The table rows are visited in `physical order`, because that is how the bitmap is laid out; this means that any ordering of the original indexes is lost, and so a separate sort step will be needed if the query has an ORDER BY clause. For this reason, and because each additional index scan adds extra time, **the planner will sometimes choose to use a simple index scan even though additional indexes are available that could have been used as well.**

In all but the simplest applications, there are various combinations of indexes that might be useful, and the database developer must make trade-offs to decide which indexes to provide. Sometimes multicolumn indexes are best, but sometimes it's better to create separate indexes and rely on the `index-combination feature.` For example, if your workload includes a mix of queries that sometimes involve only column x, sometimes only column y, and sometimes both columns, you might choose to create two separate indexes on x and y, relying on index combination to process the queries that use both columns. You could also create a multicolumn index on (x, y). This index would typically be more efficient than index combination for queries involving both columns, but as discussed in [Section 11.3](https://www.postgresql.org/docs/12/indexes-multicolumn.html), it would be almost useless for queries involving only y, so it should not be the only index. *A combination of the multicolumn index and a separate index on y would serve reasonably well*. For queries involving only x, the multicolumn index could be used, though it would be larger and hence slower than an index on x alone. The last alternative is to create all three indexes, but this is probably only reasonable if the table is searched much more often than it is updated and all three types of query are common. If one of the types of query is much less common than the others, you'd probably settle for creating just the two indexes that best match the common types.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA5MjM1OTAzMCwtMTk2MDA0NTUyNiwyMD
cyNDYzODE1LDEwODQzNDk2OTZdfQ==
-->