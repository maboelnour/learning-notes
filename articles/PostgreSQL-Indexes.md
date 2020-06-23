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


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTAzNDcxMjg4NiwyMDcyNDYzODE1LDEwOD
QzNDk2OTZdfQ==
-->