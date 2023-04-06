---
title: Index Selection
summary: Choose the best indexes for TiDB query optimization.
---

# Index Selection

Reading data from storage engines is one of the most time-consuming steps during the SQL execution. Currently, TiDB supports reading data from different storage engines and different indexes. Query execution performance depends largely on whether you select a suitable index or not.

This document introduces how to select an index to access a table, and some related ways to control index selection.

## Access tables

Before introducing index selection, it is important to understand the ways TiDB accesses tables, what triggers each way, what differences each way makes, and what the pros and cons are.

### Operators for accessing tables

| Operator | Trigger Conditions | Applicable Scenarios | Explanations |
| :------- | :------- | :------- | :---- |
| PointGet / BatchPointGet | When accessing tables in one or more single point ranges. | Any scenario | If triggered, it is usually considered as the fastest operator, since it calls the kvget interface directly to perform the calculations rather than calls the coprocessor interface.  |
| TableReader | None | Any scenario | It is generally considered as the least efficient operator that scans table data directly from the TiKV layer. It can be selected only if there is a range query on the `_tidb_rowid` column, or if there are no other operators for accessing tables to choose from. |
| TableReader | A table has a replica on the TiFlash node. | There are fewer columns to read, but many rows to evaluate.  | Tiflash is column-based storage. If you need to calculate a small number of columns and a large number of rows, it is recommended to choose this operator. |
| IndexReader | A table has one or more indexes, and the columns needed for the calculation are included in the indexes. | When there is a smaller range query on the indexes, or when there is an order requirement for indexed columns. | When multiple indexes exist, a reasonable index is selected based on the cost estimation. |
| IndexLookupReader | A table has one or more indexes, and the columns needed for calculation are not completely included in the indexes. | Same as IndexReader. |  Since the index does not completely cover calculated columns, TiDB needs to retrieve rows from a table after reading indexes. There is an extra cost compared to the IndexReader operator. |

> **Note:**
>
> The TableReader operator is based on the `_tidb_rowid` column index, and TiFlash uses a column storage index, so the selection of index is the selection of an operator for accessing tables.

## Index selection rules

TiDB selects indexes based on rules or cost. The based rules include pre-rules and skyline-pruning. When selecting an index, TiDB tries the pre-rule first. If an index satisfies a pre-rule, TiDB directly selects this index. Otherwise, TiDB uses skyline-pruning to exclude unsuitable indexes, and then selects the index with the lowest cost based on the cost estimation of each operator that accesses tables.

### Rule-based selection

#### Pre-rules

TiDB uses the following heuristic pre-rules to select indexes:

+ Rule 1: If an index satisfies "unique index with full match + no need to retrieve rows from a table (which means that the plan generated by the index is the IndexReader operator)", TiDB directly selects this index.

+ Rule 2: If an index satisfies "unique index with full match + the need to retrieve rows from a table (which means that the plan generated by the index is the IndexReader operator)", TiDB selects the index with the smallest number of rows to be retrieved from a table as a candidate index.

+ Rule 3: If an index satisfies "ordinary index + no need to retrieve rows from a table + the number of rows to be read is less than the value of a certain threshold", TiDB selects the index with the smallest number of rows to be read as a candidate index.

+ Rule 4: If only one candidate index is selected based on rule 2 and 3, select this candidate index. If two candidate indexes are respectively selected based on rule 2 and 3, select the index with the smaller number of rows to be read (the number of rows with index + the number of rows to be retrieved from a table).

The "index with full match" in the above rules means each indexed column has the equal condition. When executing the `EXPLAIN FORMAT = 'verbose' ...` statement, if the pre-rules match an index, TiDB outputs a NOTE-level warning indicating that the index matches the pre-rule.

In the following example, because the index `idx_b` meets the condition "unique index with full match + the need to retrieve rows from a table" in rule 2, TiDB selects the index `idx_b` as the access path, and `SHOW WARNING` returns a note indicating that the index  `idx_b` matches the pre-rule.

```sql
mysql> CREATE TABLE t(a INT PRIMARY KEY, b INT, c INT, UNIQUE INDEX idx_b(b));
Query OK, 0 rows affected (0.01 sec)

mysql> EXPLAIN FORMAT = 'verbose' SELECT b, c FROM t WHERE b = 3 OR b = 6;
+-------------------+---------+---------+------+-------------------------+------------------------------+
| id                | estRows | estCost | task | access object           | operator info                |
+-------------------+---------+---------+------+-------------------------+------------------------------+
| Batch_Point_Get_5 | 2.00    | 8.80    | root | table:t, index:idx_b(b) | keep order:false, desc:false |
+-------------------+---------+---------+------+-------------------------+------------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS;
+-------+------+-------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                   |
+-------+------+-------------------------------------------------------------------------------------------+
| Note  | 1105 | unique index idx_b of t is selected since the path only has point ranges with double scan |
+-------+------+-------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### Skyline-pruning

Skyline-pruning is a heuristic filtering rule for indexes, which can reduce the probability of wrong index selection caused by wrong estimation. To judge an index, the following three dimensions are needed:

- How many access conditions are covered by the indexed columns. An “access condition” is a where condition that can be converted to a column range. And the more access conditions an indexed column set covers, the better it is in this dimension.

- Whether it needs to retrieve rows from a table when you select the index to access the table (that is, the plan generated by the index is IndexReader operator or IndexLookupReader operator). Indexes that do not retrieve rows from a table are better on this dimension than indexes that do. If both indexes need TiDB to retrieve rows from the table, compare how many filtering conditions are covered by the indexed columns. Filtering conditions mean the `where` condition that can be judged based on the index. If the column set of an index covers more access conditions, the smaller the number of retrieved rows from a table, and the better the index is in this dimension.

- Select whether the index satisfies a certain order. Because index reading can guarantee the order of certain column sets, indexes that satisfy the query order are superior to indexes that do not satisfy on this dimension.

For these three dimensions above, if the index `idx_a` performs no worse than the index `idx_b` in all three dimensions and performs better than `idx_b` in one dimension, then `idx_a` is preferred. When executing the `EXPLAIN FORMAT = 'verbose' ...` statement, if skyline-pruning excludes some indexes, TiDB outputs a NOTE-level warning listing the remaining indexes after the skyline-pruning exclusion.

In the following example, the indexes `idx_b` and `idx_e` are both inferior to `idx_b_c`, so they are excluded by skyline-pruning. The returned result of `SHOW WARNING` displays the remaining indexes after skyline-pruning.

```sql
mysql> CREATE TABLE t(a INT PRIMARY KEY, b INT, c INT, d INT, e INT, INDEX idx_b(b), INDEX idx_b_c(b, c), INDEX idx_e(e));
Query OK, 0 rows affected (0.01 sec)

mysql> EXPLAIN FORMAT = 'verbose' SELECT * FROM t WHERE b = 2 AND c > 4;
+-------------------------------+---------+---------+-----------+------------------------------+----------------------------------------------------+
| id                            | estRows | estCost | task      | access object                | operator info                                      |
+-------------------------------+---------+---------+-----------+------------------------------+----------------------------------------------------+
| IndexLookUp_10                | 33.33   | 738.29  | root      |                              |                                                    |
| ├─IndexRangeScan_8(Build)     | 33.33   | 2370.00 | cop[tikv] | table:t, index:idx_b_c(b, c) | range:(2 4,2 +inf], keep order:false, stats:pseudo |
| └─TableRowIDScan_9(Probe)     | 33.33   | 2370.00 | cop[tikv] | table:t                      | keep order:false, stats:pseudo                     |
+-------------------------------+---------+---------+-----------+------------------------------+----------------------------------------------------+
3 rows in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS;
+-------+------+------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                  |
+-------+------+------------------------------------------------------------------------------------------+
| Note  | 1105 | [t,idx_b_c] remain after pruning paths for t given Prop{SortItems: [], TaskTp: rootTask} |
+-------+------+------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### Cost estimation-based selection

After using the skyline-pruning rule to rule out inappropriate indexes, the selection of indexes is based entirely on the cost estimation. The cost estimation of accessing tables requires the following considerations:

- The average length of each row of the indexed data in the storage engine.
- The number of rows in the query range generated by the index.
- The cost for retrieving rows from a table.
- The number of ranges generated by index during the query execution.

According to these factors and the cost model, the optimizer selects an index with the lowest cost to access the table.

#### Common tuning problems with cost estimation based selection

1. The estimated number of rows is not accurate?

    This is usually due to stale or inaccurate statistics. You can re-execute the `analyze table` statement or modify the parameters of the `analyze table` statement.

2. Statistics are accurate, and reading from TiFlash is faster, but why does the optimizer choose to read from TiKV?

    At present, the cost model of distinguishing TiFlash from TiKV is still rough. You can decrease the value of `tidb_opt_seek_factor` parameter, then the optimizer prefers to choose TiFlash.

3. The statistics are accurate. Index A needs to retrieve rows from tables, but it actually executes faster than Index B that does not retrieve rows from tables. Why does the optimizer choose Index B?

    In this case, the cost estimation may be too large for retrieving rows from tables. You can decrease the value of `tidb_opt_network_factor` parameter to reduce the cost of retrieving rows from tables.

## Control index selection

The index selection can be controlled by a single query through [Optimizer Hints](/optimizer-hints.md).

- `USE_INDEX` / `IGNORE_INDEX` can force the optimizer to use / not use certain indexes.

- `READ_FROM_STORAGE` can force the optimizer to choose the TiKV / TiFlash storage engine for certain tables to execute queries.