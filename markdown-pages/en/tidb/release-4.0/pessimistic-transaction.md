---
title: TiDB Pessimistic Transaction Model
summary: Learn the pessimistic transaction model in TiDB.
aliases: ['/docs/stable/pessimistic-transaction/','/docs/v4.0/pessimistic-transaction/','/docs/stable/reference/transactions/transaction-pessimistic/']
---

# TiDB Pessimistic Transaction Model

To make the usage of TiDB closer to traditional databases and reduce the cost of migration, starting from v3.0, TiDB supports the pessimistic transaction model on top of the optimistic transaction model. This document describes the features of the TiDB pessimistic transaction model.

> **Note:**
>
> Starting from v3.0.8, newly created TiDB clusters use the pessimistic transaction model by default. However, this does not affect your existing cluster if you upgrade it from v3.0.7 or earlier to v3.0.8 or later. In other words, **only newly created clusters default to using the pessimistic transaction model**.

## Switch transaction mode

You can set the transaction mode by configuring the [`tidb_txn_mode`](/system-variables.md#tidb_txn_mode) system variable. The following command sets all explicit transactions (that is, non-autocommit transactions) executed by newly created sessions in the cluster to the pessimistic transaction mode:


```sql
SET GLOBAL tidb_txn_mode = 'pessimistic';
```

You can also explicitly enable the pessimistic transaction mode by executing the following SQL statements:


```sql
BEGIN PESSIMISTIC;
```


```sql
BEGIN /*T! PESSIMISTIC */;
```

The `BEGIN PESSIMISTIC;` and `BEGIN OPTIMISTIC;` statements take precedence over the `tidb_txn_mode` system variable. Transactions started with these two statements ignore the system variable and support using both the pessimistic and optimistic transaction modes.

## Behaviors

Pessimistic transactions in TiDB behave similarly to those in MySQL. See the minor differences in [Difference with MySQL InnoDB](#difference-with-mysql-innodb).

- When you execute `UPDATE`, `DELETE` or `INSERT` statements, the **latest** committed data is read, data is modified, and a pessimistic lock is applied on the modified rows.

- For `SELECT FOR UPDATE` statements, a pessimistic lock is applied on the latest version of the committed data, instead of on the modified rows.

- Locks will be released when the transaction is committed or rolled back. Other transactions attempting to modify the data are blocked and have to wait for the lock to be released. Transactions attempting to _read_ the data are not blocked, because TiDB uses multi-version concurrency control (MVCC).

- If several transactions are trying to acquire each other's respective locks, a deadlock will occur. This is automatically detected, and one of the transactions will randomly be terminated with a MySQL-compatible error code `1213` returned.

- Transactions will wait up to `innodb_lock_wait_timeout` seconds (default: 50) to acquire new locks. When this timeout is reached, a MySQL-compatible error code `1205` is returned. If multiple transactions are waiting for the same lock, the order of priority is approximately based on the `start ts` of the transaction.

- TiDB supports both the optimistic transaction mode and pessimistic transaction mode in the same cluster. You can specify either mode for transaction execution.

- TiDB supports the `FOR UPDATE NOWAIT` syntax and does not block and wait for locks to be released. Instead, a MySQL-compatible error code `3572` is returned.

- If the `Point Get` and `Batch Point Get` operators do not read data, they still lock the given primary key or unique key, which blocks other transactions from locking or writing data to the same primary key or unique key.

## Difference with MySQL InnoDB

1. When TiDB executes DML or `SELECT FOR UPDATE` statements that use range in the WHERE clause, concurrent DML statements within the range are not blocked.
    
    For example:
    
    ```sql
    CREATE TABLE t1 (
     id INT NOT NULL PRIMARY KEY,
     pad1 VARCHAR(100)
    );
    INSERT INTO t1 (id) VALUES (1),(5),(10);
    ```
    
    ```sql
    BEGIN /*T! PESSIMISTIC */;
    SELECT * FROM t1 WHERE id BETWEEN 1 AND 10 FOR UPDATE;
    ```
    
    ```sql
    BEGIN /*T! PESSIMISTIC */;
    INSERT INTO t1 (id) VALUES (6); -- blocks only in MySQL
    UPDATE t1 SET pad1='new value' WHERE id = 5; -- blocks waiting in both MySQL and TiDB
    ```
    
    This behavior is because TiDB does not currently support _gap locking_.

2. TiDB does not support `SELECT LOCK IN SHARE MODE`.

    When `SELECT LOCK IN SHARE MODE` is executed, it has the same effect as that without the lock, so the read or write operation of other transactions is not blocked.

3. DDL may result in failure of the pessimistic transaction commit.

    When DDL is executed in MySQL, it might be blocked by the transaction that is being executed. However, in this scenario, the DDL operation is not blocked in TiDB, which leads to failure of the pessimistic transaction commit: `ERROR 1105 (HY000): Information schema is changed. [try again later]`. TiDB executes the `TRUNCATE TABLE` statement during the transaction execution, which might result in the `table doesn't exist` error.

4. After executing `START TRANSACTION WITH CONSISTENT SNAPSHOT`, MySQL can still read the tables that are created later in other transactions, while TiDB cannot.

5. The autocommit transactions prefer the optimistic locking.

    When using the pessimistic model, the autocommit transactions first try to commit the statement using the optimistic model that has less overhead. If a write conflict occurs, the pessimistic model is used for transaction retry. Therefore, if `tidb_retry_limit` is set to `0`, the autocommit transaction still reports the `Write Conflict` error when a write conflict occurs.

    The autocommit `SELECT FOR UPDATE` statement does not wait for lock.

6. The data read by `EMBEDDED SELECT` in the statement is not locked.

7. Open transactions in TiDB do not block garbage collection (GC). By default, this limits the maximum execution time of pessimistic transactions to 10 minutes. You can modify this limit by editing `max-txn-ttl` under `[performance]` in the TiDB configuration file.

## Isolation level

TiDB supports the following two isolation levels in the pessimistic transaction mode:

- [Repeatable Read](/transaction-isolation-levels.md#repeatable-read-isolation-level) by default, which is the same as MySQL.

    > **Note:**
    >
    > In this isolation level, DML operations are performed based on the latest committed data. The behavior is the same as MySQL, but differs from the optimistic transaction mode in TiDB. See [Difference between TiDB and MySQL Repeatable Read](/transaction-isolation-levels.md#difference-between-tidb-and-mysql-repeatable-read).

- [Read Committed](/transaction-isolation-levels.md#read-committed-isolation-level). You can set this isolation level using the [`SET TRANSACTION`](/sql-statements/sql-statement-set-transaction.md) statement.

## Pipelined locking process

Adding a pessimistic lock requires writing data into TiKV. The response of successfully adding a lock can only be returned to TiDB after commit and apply through Raft. Therefore, compared with optimistic transactions, the pessimistic transaction mode inevitably has higher latency.

To reduce the overhead of locking, TiKV implements the pipelined locking process: when the data meets the requirements for locking, TiKV immediately notifies TiDB to execute subsequent requests and writes into the pessimistic lock asynchronously. This process reduces most latency and significantly improves the performance of pessimistic transactions. However, when network partition occurs in TiKV or a TiKV node is down, the asynchronous write into the pessimistic lock might fail and affect the following aspects:

* Other transactions that modify the same data cannot be blocked. If the application logic relies on locking or lock waiting mechanisms, the correctness of the application logic is affected.

* There is a low probability that the transaction commit fails, but it does not affect the correctness of the transactions.

If the application logic relies on the locking or lock waiting mechanisms, or if you want to guarantee as much as possible the success rate of transaction commits even in the case of TiKV cluster anomalies, you should disable the pipelined locking feature.

![Pipelined pessimistic lock](https://download.pingcap.com/images/docs/pessimistic-transaction-pipelining.png)

This feature is disabled by default. To enable it, modify the TiKV configuration:

```toml
[pessimistic-txn]
pipelined = true
```

If the TiKV cluster is v4.0.9 or later, you can also dynamically enable this feature by [modifying TiKV configuration online](/dynamic-config.md#modify-tikv-configuration-online):


```sql
set config tikv pessimistic-txn.pipelined='true';
```

## FAQ

1. The TiDB log shows `pessimistic write conflict, retry statement`.

    When a write conflict occurs, the optimistic transaction is terminated directly, but the pessimistic transaction retries the statement with the latest data until there is no write conflict. The log prints this entry with each retry, so there is no need for extra attention.

2. When DML is executed, an error `pessimistic lock retry limit reached` is returned.

    In the pessimistic transaction mode, every statement has a retry limit. This error is returned when the retry times of write conflict exceeds the limit. The default retry limit is `256`. To change the limit, modify the `max-retry-limit` under the `[pessimistic-txn]` category in the TiDB configuration file.

3. The execution time limit for pessimistic transactions.

    In TiDB 4.0, garbage collection (GC) does not affect the running transactions, but the execution time of pessimistic transactions cannot exceed 10 minutes by default. You can modify this limit by editing `max-txn-ttl` under `[performance]` in the TiDB configuration file.