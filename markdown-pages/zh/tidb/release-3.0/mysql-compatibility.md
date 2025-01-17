---
title: 与 MySQL 兼容性对比
aliases: ['/docs-cn/v3.0/mysql-compatibility/','/docs-cn/v3.0/reference/mysql-compatibility/']
---

# 与 MySQL 兼容性对比

TiDB 支持 MySQL 传输协议及其绝大多数的语法。这意味着您现有的 MySQL 连接器和客户端都可以继续使用。大多数情况下您现有的应用都可以迁移至 TiDB，无需任何代码修改。

当前 TiDB 服务器官方支持的版本为 MySQL 5.7。大部分 MySQL 运维工具（如 PHPMyAdmin, Navicat, MySQL Workbench 等），以及备份恢复工具（如 mysqldump, Mydumper/myloader）等都可以直接使用。

但 TiDB 尚未支持一些 MySQL 功能，可能的原因如下：

- 有更好的解决方案，例如 JSON 取代 XML 函数。
- 目前对这些功能的需求度不高，例如存储流程和函数。
- 一些功能在分布式系统上的实现难度较大。

> **注意：**
>
> 本页内容仅涉及 MySQL 与 TiDB 的总体差异。关于[安全特性](/security-compatibility-with-mysql.md)、[悲观事务模型](/pessimistic-transaction.md#和-mysql-innodb-的差异)的兼容信息请查看各自具体页面。

## 不支持的特性

* 存储过程与函数
* 触发器
* 事件
* 自定义函数
* 外键约束 [#18209](https://github.com/pingcap/tidb/issues/18209)
* 临时表 [#1248](https://github.com/pingcap/tidb/issues/1248)
* 全文/空间函数与索引 [#1793](https://github.com/pingcap/tidb/issues/1793)
* 非 `ascii`/`latin1`/`binary`/`utf8`/`utf8mb4` 的字符集
* `BINARY` 之外的排序规则
* 增加/删除主键
* SYS schema
* MySQL 追踪优化器
* XML 函数
* X-Protocol [#1109](https://github.com/pingcap/tidb/issues/1109)
* Savepoints [#6840](https://github.com/pingcap/tidb/issues/6840)
* 列级权限 [#9766](https://github.com/pingcap/tidb/issues/9766)
* `XA` 语法（TiDB 内部使用两阶段提交，但并没有通过 SQL 接口公开）
* `CREATE TABLE tblName AS SELECT stmt` 语法 [#4754](https://github.com/pingcap/tidb/issues/4754)
* `CHECK TABLE` 语法 [#4673](https://github.com/pingcap/tidb/issues/4673)
* `CHECKSUM TABLE` 语法 [#1895](https://github.com/pingcap/tidb/issues/1895)
* `GET_LOCK` 和 `RELEASE_LOCK` 函数 [#14994](https://github.com/pingcap/tidb/issues/14994)

## 与 MySQL 有差异的特性

### 自增 ID

TiDB 中，自增列只保证自增且唯一，也能保证在单个 TiDB server 中自增，但不保证多个 TiDB server 中自增，不保证连续分配。TiDB 目前采用批量分配 ID 的方式，所以如果在多台 TiDB 上同时插入数据，分配的自增 ID 会不连续。TiDB 可通过 `tidb_allow_remove_auto_inc` 系统变量开启或者关闭删除列的 `AUTO_INCREMENT` 属性。删除列属性的语法是：`alter table modify` 或 `alter table change`。

在集群中有多个 tidb-server 实例时，如果表结构中有自增 ID，建议不要混用缺省值和自定义值，否则在如下情况下会遇到问题。

假设有这样一个带有自增 ID 的表：


```sql
create table t(id int unique key AUTO_INCREMENT, c int);
```

TiDB 实现自增 ID 的原理是每个 tidb-server 实例缓存一段 ID 值用于分配（目前会缓存 30000 个 ID），用完这段值再去取下一段。

假设集群中有两个 tidb-server 实例 A 和 B（A 缓存 [1,30000] 的自增 ID，B 缓存 [30001,60000] 的自增 ID），依次执行如下操作：

1. 客户端向 B 插入一条将 `id` 设置为 1 的语句 `insert into t values (1, 1)`，并执行成功。
2. 客户端向 A 发送 Insert 语句 `insert into t (c) (1)`，这条语句中没有指定 `id` 的值，所以会由 A 分配，当前 A 缓存了 [1, 30000] 这段 ID，所以会分配 1 为自增 ID 的值，并把本地计数器加 1。而此时数据库中已经存在 `id` 为 1 的数据，最终返回 `Duplicated Error` 错误。

另外，从 TiDB 3.0.4 版本开始，TiDB 将通过系统变量 `@@tidb_allow_remove_auto_inc` 控制是否允许通过 `alter table modify` 或 `alter table change` 来移除列的 `AUTO_INCREMENT` 属性，默认是不允许移除。移除后不可再恢复（因为 TiDB 不支持添加列的 `AUTO_INCREMENT` 属性）

> **注意：**
>
> 在没有指定主键的情况下 TiDB 会使用 `_tidb_rowid` 来标识行，该数值的分配会和自增列（如果存在的话）共用一个分配器。如果指定了自增列为主键，则 TiDB 会用该列来标识行。因此会有以下的示例情况：

```sql
mysql> create table t(id int unique key AUTO_INCREMENT);
Query OK, 0 rows affected (0.05 sec)

mysql> insert into t values(),(),();
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> select _tidb_rowid, id from t;
+-------------+------+
| _tidb_rowid | id   |
+-------------+------+
|           4 |    1 |
|           5 |    2 |
|           6 |    3 |
+-------------+------+
3 rows in set (0.01 sec)
```

TiDB 自增 ID 的缓存大小在早期版本中是对用户透明的。从 v3.1.2、v3.0.14 和 v4.0.rc.2 版本开始，TiDB 引入了 `AUTO_ID_CACHE` 表选项来允许用户自主设置自增 ID 分配缓存的大小。其中缓存大小可能会被自增列和 `_tidb_rowid` 共同消耗。此外如果在 `INSERT` 语句中所需连续 ID 长度超过 `AUTO_ID_CACHE` 的长度时，TiDB 会适当调大缓存以便能够保证该语句的正常插入。

### Performance schema

Performance schema 表在 TiDB 中返回结果为空。TiDB 使用 [Prometheus 和 Grafana](/monitor-a-tidb-cluster.md) 来监测性能指标。

从 TiDB 3.0.4 版本开始，TiDB 支持 `events_statements_summary_by_digest`，参见 [Statement Summary Table](/statement-summary-tables.md)。

### 查询计划

TiDB 的查询计划（`EXPLAIN`/`EXPLAIN FOR`）输出格式与 MySQL 差别较大，同时 `EXPLAIN FOR` 的输出内容与权限设置与 MySQL 不一致，参见[理解 TiDB 执行计划](/query-execution-plan.md)。

### 内建函数

TiDB 支持常用的 MySQL 内建函数，但是不是所有的函数都已经支持，具体请参考[语法文档](https://pingcap.github.io/sqlgram/#functioncallkeyword)。

### DDL

在 TiDB 中，运行的 DDL 操作不会影响对表的读取或写入。但是，目前 DDL 变更有如下一些限制：

+ Add Index
    - 不支持同时创建多个索引
    - 不支持 `VISIBLE/INVISIBLE` 的索引
    - 不支持通过 `ALTER TABLE` 在[生成列](/generated-columns.md)上添加索引
    - 其他类型的 Index Type (HASH/BTREE/RTREE) 只有语法支持，功能不支持
+ Add Column
    - 不支持同时创建多个列
    - 不支持将新创建的列设为主键或唯一索引，也不支持将此列设成 AUTO_INCREMENT 属性
+ Drop Column: 不支持删除主键列或索引列
+ Change/Modify Column
    + 不支持有损变更，比如从 `BIGINT` 变为 `INTEGER`，或者从 `VARCHAR(255)` 变为 `VARCHAR(10)`，否则可能输出的错误信息 `length %d is less than origin %d`。
    + 不支持修改 `DECIMAL` 类型的精度
    + 不支持将字段类型修改为其超集，例如不支持从 `INTEGER` 修改为 `VARCHAR`，或者从 `TIMESTAMP` 修改为 `DATETIME`，否则可能输出的错误信息 `Unsupported modify column: type %d not match origin %d`。
    + 不支持更改 `UNSIGNED` 属性
    + 只支持将 `CHARACTER SET` 属性从 `utf8` 更改为 `utf8mb4`
+ Alter Database
    + 只支持将 `CHARACTER SET` 属性从 `utf8` 更改为 `utf8mb4`
+ `LOCK [=] {DEFAULT|NONE|SHARED|EXCLUSIVE}`: TiDB 支持的语法，但是在 TiDB 中不会生效。所有支持的 DDL 变更都不会锁表。
+ `ALGORITHM [=] {DEFAULT|INSTANT|INPLACE|COPY}`: TiDB 完全支持 `ALGORITHM=INSTANT` 和 `ALGORITHM=INPLACE` 语法，但运行过程与 MySQL 有所不同，因为 MySQL 中的一些 `INPLACE` 操作实际上是 TiDB 中的 `INSTANT` 操作。`ALGORITHM=COPY` 语法在 TiDB 中不会生效，会返回警告信息。
+ 单个 `ALTER TABLE` 语句中无法完成多个操作。例如，不能用一个语句来添加多个列或多个索引。
+ Table Option 不支持以下语法
    - `WITH/WITHOUT VALIDATION`
    - `SECONDARY_LOAD/SECONDARY_UNLOAD`
    - `CHECK/DROP CHECK`
    - `STATS_AUTO_RECALC/STATS_SAMPLE_PAGES`
    - `SECONDARY_ENGINE`
    - `ENCRYPTION`
+ Table Partition 不支持以下语法
    - `PARTITION BY LIST`
    - `PARTITION BY KEY`
    - `SUBPARTITION`
    - `{CHECK|EXCHANGE|OPTIMIZE|REPAIR|IMPORT|DISCARD|REBUILD|REORGANIZE|COALESCE} PARTITION`

### `ANALYZE TABLE`

- [`ANALYZE TABLE`](/statistics.md#手动收集) 语句在 TiDB 和 MySQL 中表现不同。在 MySQL/InnoDB 中，它是一个轻量级语句，执行过程较短；而在 TiDB 中，它会完全重构表的统计数据，语句执行过程较长。

### `SELECT` 的限制

- 不支持 `SELECT ... INTO @变量` 语法。
- 不支持 `SELECT ... GROUP BY ... WITH ROLLUP` 语法。
- TiDB 中的 `SELECT .. GROUP BY expr` 的返回结果与 MySQL 5.7 并不一致。MySQL 5.7 的结果等价于 `GROUP BY expr ORDER BY expr`。而 TiDB 中该语法所返回的结果并不承诺任何顺序，与 MySQL 8.0 的行为一致。

### 视图

目前 TiDB 不支持对视图进行 `UPDATE`、`INSERT`、`DELETE` 等写入操作。

### 存储引擎

出于兼容性原因，TiDB 支持使用备用存储引擎创建表的语法。元数据命令将表描述为 InnoDB 存储引擎：


```sql
CREATE TABLE t1 (a INT) ENGINE=MyISAM;
```

```
Query OK, 0 rows affected (0.14 sec)
```


```sql
SHOW CREATE TABLE t1;
```

```
*************************** 1. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `a` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
1 row in set (0.00 sec)
```

从架构上讲，TiDB 确实支持类似 MySQL 的存储引擎抽象，在启动 TiDB（通常是 `tikv`）时 [`--store`](/command-line-flags-for-tidb-configuration.md#--store) 选项指定的引擎中创建用户表。

### SQL 模式

TiDB 支持大部分 [SQL 模式](/sql-mode.md)。不支持的 SQL 模式如下：

- 不支持兼容模式，例如：`ORACLE` 和 `POSTGRESQL`（TiDB 解析但会忽略这两个兼容模式），MySQL 5.7 已弃用兼容模式，MySQL 8.0 已移除兼容模式。
- TiDB 的 `ONLY_FULL_GROUP_BY` 模式与 MySQL 5.7 相比有细微的[语义差别](/functions-and-operators/aggregate-group-by-functions.md#与-mysql-的区别)。

MySQL 中的 `NO_DIR_IN_CREATE` 和 `NO_ENGINE_SUBSTITUTION` 的 SQL 模式可用于解决兼容性问题，并不适用于 TiDB。

### 默认设置的区别

+ 默认字符集与 MySQL 不同：
    + TiDB 中为 `utf8mb4`
    + MySQL 5.7 中为 `latin1`，MySQL 8.0 中修改为 `utf8mb4`
+ 默认排序规则不同：
    + TiDB 中，`utf8mb4` 的默认排序规则为 `utf8mb4_bin`
    + MySQL 5.7 中，`utf8mb4` 的默认排序规则为 `utf8mb4_general_ci`，MySQL 8.0 中修改为 `utf8mb4_0900_ai_ci`
    + 请使用 [`SHOW CHARACTER SET`](/sql-statements/sql-statement-show-character-set.md) 语句查看所有字符集的默认排序规则
+ `foreign_key_checks` 的默认值不同：
    + TiDB 中该值默认为 `OFF`，并且目前 TiDB 只支持设置该值为 `OFF`。
    + MySQL 5.7 中该值默认为 `ON`。
+ 默认 SQL mode 与 MySQL 5.7 相同，与 MySQL 8.0 不同：
    + TiDB 中为 `ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION`
    + MySQL 中默认设置：
        + MySQL 5.7 的默认 SQL mode 与 TiDB 相同
        + MySQL 8.0 中为 `ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION`
+ `lower_case_table_names` 的默认值不同：
    + TiDB 中该值默认为 2，并且目前 TiDB 只支持设置该值为 2
    + MySQL 中默认设置：
        + Linux 系统中该值为 0
        + Windows 系统中该值为 1
        + macOS 系统中该值为 2
+ `explicit_defaults_for_timestamp` 的默认值不同：
    + TiDB 中该值默认为 `ON`，并且目前 TiDB 只支持设置该值为 `ON`
    + MySQL 中默认设置：
        + MySQL 5.7：`OFF`
        + MySQL 8.0：`ON`

### 日期时间处理的区别

#### 时区

MySQL 默认使用本地时区，依赖于系统内置的当前的时区规则（例如什么时候开始夏令时等）进行计算；且在未[导入时区表数据](https://dev.mysql.com/doc/refman/8.0/en/time-zone-support.html#time-zone-installation)的情况下不能通过时区名称来指定时区。

TiDB 不需要导入时区表数据也能使用所有时区名称，采用系统当前安装的所有时区规则进行计算（一般为 `tzdata` 包），且无法通过导入时区表数据的形式修改计算规则。

> **注意：**
>
> 能下推到 TiKV 的时间相关表达式会由 TiKV 进行计算。TiKV 总是采用 TiKV 内置时区规则计算，而不依赖于系统所安装的时区规则。若系统安装的时区规则与 TiKV 内置的时区规则版本不匹配，则在少数情况下可能发生能插入的时间数据无法再读出来的问题。例如，若系统上安装了 tzdata 2018a 时区规则，则在时区设置为 Asia/Shanghai 或时区设置为本地时区且本地时区为 Asia/Shanghai 的情况下，时间 `1988-04-17 02:00:00` 可以被正常插入 TiDB 3.0 RC.1，但该记录对于特定类型 SQL 则无法再读出来，原因是 TiKV 3.0 RC.1 依据的 tzdata 2018i 规则中该时间在 Asia/Shanghai 时区中不存在（夏令时时间后移一小时）。
>
> TiKV 各个版本内置的时区规则如下：
>
> - 3.0.0 RC.1 及以后：[tzdata 2018i](https://github.com/eggert/tz/tree/2018i)
> - 2.1.0 RC.1 及以后：[tzdata 2018e](https://github.com/eggert/tz/tree/2018e)

#### 零月和零日

与 MySQL 一样，TiDB 默认启用了 `NO_ZERO_DATE` 和 `NO_ZERO_IN_DATE` 模式，不建议将这两个模式设为禁用。尽管将这些模式设为禁用时 TiDB 仍可正常使用，但 TiKV coprocessor 会受到影响，具体表现为，执行特定类型的语句，将日期和时间处理函数下推到 TiKV 时可能会导致语句错误。

#### 字符串类型行末空格的处理

目前 TiDB 在进行数据插入时，对于 `VARCHAR` 类型会保留行末空格，对于 `CHAR` 类型会插入截断空格后的数据。在没有索引的情况下，TiDB 和 MySQL 行为保持一致。如果 `VARCHAR` 类型上有 `UNIQUE` 索引，MySQL 在判断是否重复的时候，和处理 `CHAR` 类型一样，先截断 `VARCHAR` 数据末行空格再作判断；TiDB 则是按照保留空格的情况处理。

在做比较时，MySQL 会先截去常量和 Column 的末尾空格再作比较，而 TiDB 则是保留常量和 Column 的末尾空格来做精确比较。

### 类型系统的区别

以下的列类型 MySQL 支持，但 TiDB 不支持：

+ FLOAT4/FLOAT8
+ FIXED (alias for DECIMAL)
+ SERIAL (alias for BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE)
+ SQL_TSI_* （包括 SQL_TSI_YEAR、SQL_TSI_MONTH、SQL_TSI_WEEK、SQL_TSI_DAY、SQL_TSI_HOUR、SQL_TSI_MINUTE 和 SQL_TSI_SECOND）
