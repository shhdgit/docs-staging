---
title: SELECT | TiDB SQL Statement Reference
summary: An overview of the usage of SELECT for the TiDB database.
---

# 選択する {#select}

`SELECT`ステートメントは、TiDB からデータを読み取るために使用されます。

## あらすじ {#synopsis}

**選択Stmt:**

![SelectStmt](https://download.pingcap.com/images/docs/sqlgram/SelectStmt.png)

<CustomContent platform="tidb-cloud">

> **ノート：**
>
> `SELECT ... INTO OUTFILE`ステートメントはTiDB Cloudではサポートされていません。

</CustomContent>

**デュアルから:**

![FromDual](https://download.pingcap.com/images/docs/sqlgram/FromDual.png)

**StmtOpts を選択:**

![SelectStmtOpts](https://download.pingcap.com/images/docs/sqlgram/SelectStmtOpts.png)

**選択StmtFieldList:**

![SelectStmtFieldList](https://download.pingcap.com/images/docs/sqlgram/SelectStmtFieldList.png)

**TableRefsClause:**

```ebnf+diagram
TableRefsClause ::=
    TableRef AsOfClause? ( ',' TableRef AsOfClause? )*

AsOfClause ::=
    'AS' 'OF' 'TIMESTAMP' Expression
```

**WhereClauseオプション:**

![WhereClauseOptional](https://download.pingcap.com/images/docs/sqlgram/WhereClauseOptional.png)

**選択StmtGroup:**

![SelectStmtGroup](https://download.pingcap.com/images/docs/sqlgram/SelectStmtGroup.png)

**所有条項:**

![HavingClause](https://download.pingcap.com/images/docs/sqlgram/HavingClause.png)

**OrderByオプション:**

![OrderByOptional](https://download.pingcap.com/images/docs/sqlgram/OrderByOptional.png)

**選択StmtLimit:**

![SelectStmtLimit](https://download.pingcap.com/images/docs/sqlgram/SelectStmtLimit.png)

**最初または次:**

![FirstOrNext](https://download.pingcap.com/images/docs/sqlgram/FirstOrNext.png)

**FetchFirstOpt:**

![FetchFirstOpt](https://download.pingcap.com/images/docs/sqlgram/FetchFirstOpt.png)

**行または行:**

![RowOrRows](https://download.pingcap.com/images/docs/sqlgram/RowOrRows.png)

**ロックオプションを選択:**

```ebnf+diagram
SelectLockOpt ::= 
    ( ( 'FOR' 'UPDATE' ( 'OF' TableList )? 'NOWAIT'? )
|   ( 'LOCK' 'IN' 'SHARE' 'MODE' ) )?

TableList ::=
    TableName ( ',' TableName )*
```

**WindowClauseオプション**

![WindowClauseOptional](https://download.pingcap.com/images/docs/sqlgram/WindowClauseOptional.png)

**テーブルサンプルオプション**

```ebnf+diagram
TableSampleOpt ::=
    'TABLESAMPLE' 'REGIONS()'
```

## 構文要素の説明 {#description-of-the-syntax-elements}

| 構文要素                           | 説明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| :----------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TableOptimizerHints`          | これは、TiDB のオプティマイザーの動作を制御するためのヒントです。詳細については、 [オプティマイザーのヒント](/optimizer-hints.md)を参照してください。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `ALL` `DISTINCT` `DISTINCTROW` | `ALL` 、 `DISTINCT` / `DISTINCTROW`修飾子は、重複した行を返すかどうかを指定します。 ALL (デフォルト) は、一致するすべての行が返されることを指定します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `HIGH_PRIORITY`                | `HIGH_PRIORITY`指定すると、現在のステートメントに他のステートメントよりも高い優先順位が与えられます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `SQL_CALC_FOUND_ROWS`          | TiDB はこの機能をサポートしていないため、 [`tidb_enable_noop_functions=1`](/system-variables.md#tidb_enable_noop_functions-new-in-v40)が設定されていない場合はエラーを返します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `SQL_CACHE` `SQL_NO_CACHE`     | `SQL_CACHE`と`SQL_NO_CACHE` 、リクエスト結果を TiKV (RocksDB) の`BlockCache`にキャッシュするかどうかを制御するために使用されます。 `count(*)`クエリなどの大量のデータに対する 1 回限りのクエリの場合は、 `BlockCache`のホット ユーザー データがフラッシュされるのを避けるために`SQL_NO_CACHE`を入力することをお勧めします。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `STRAIGHT_JOIN`                | `STRAIGHT_JOIN`を指定すると、オプティマイザは`FROM`句で使用されるテーブルの順序でユニオン クエリを実行します。オプティマイザが不適切な結合順序を選択した場合、この構文を使用してクエリの実行を高速化できます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `select_expr`                  | 各`select_expr`は取得する列を示します。列名や式も含まれます。 `\*`すべての列を表します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `FROM table_references`        | `FROM table_references`句は、行を取得するテーブル ( `select * from t;`など)、またはテーブル ( `select * from t1 join t2;`など)、または 0 テーブル ( `select 1+1;`と同等の`select 1+1 from dual;`など) を示します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `WHERE where_condition`        | `WHERE`句を指定した場合、行が選択されるために満たさなければならない条件を示します。結果には、条件を満たすデータのみが含まれます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `GROUP BY`                     | `GROUP BY`ステートメントは、結果セットをグループ化するために使用されます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `HAVING where_condition`       | `HAVING`句と`WHERE`句はどちらも結果をフィルタリングするために使用されます。 `HAVING`句は`GROUP BY`の結果をフィルタリングし、 `WHERE`句は集計の前に結果をフィルタリングします。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `ORDER BY`                     | `ORDER BY`句は、 `select_expr`リスト内の列、式、または項目に基づいてデータを昇順または降順に並べ替えるのに使用されます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `LIMIT`                        | `LIMIT`句を使用して行数を制限できます。 `LIMIT` 1 つまたは 2 つの数値引数を取ります。引数が 1 つある場合、その引数は返す行の最大数を指定します。デフォルトでは、最初に返される行はテーブルの最初の行になります。 2 つの引数を使用する場合、最初の引数は返す最初の行のオフセットを指定し、2 番目の引数は返す行の最大数を指定します。 TiDB は`FETCH FIRST/NEXT n ROW/ROWS ONLY`構文もサポートしており、これは`LIMIT n`と同じ効果があります。この構文では`n`を省略でき、その効果は`LIMIT 1`と同じです。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `Window window_definition`     | これはウィンドウ関数の構文であり、通常は分析計算を行うために使用されます。詳細については、 [窓関数](/functions-and-operators/window-functions.md)を参照してください。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `FOR UPDATE`                   | `SELECT FOR UPDATE`句は、結果セット内のすべてのデータをロックして、他のトランザクションからの同時更新を検出します。現在のトランザクションの開始後に他のトランザクションによって書き込まれた行データなど、クエリ条件には一致するが結果セットに存在しないデータは読み取りロックされません。 TiDB が[オプティミスティックトランザクションモード](/optimistic-transaction.md)使用する場合、トランザクションの競合はステートメントの実行フェーズでは検出されません。したがって、PostgreSQL などの他のデータベースのように、現在のトランザクションは他のトランザクションの実行をブロックしません`UPDATE` 、 `DELETE` 、または`SELECT FOR UPDATE` 。コミット フェーズでは、 `SELECT FOR UPDATE`によって読み取られた行は 2 つのフェーズでコミットされます。これは、これらの行も競合検出に参加できることを意味します。書き込み競合が発生した場合、 `SELECT FOR UPDATE`句を含むすべてのトランザクションのコミットは失敗します。競合が検出されなかった場合、コミットは成功します。また、ロックされた行に対して新しいバージョンが生成されるため、コミットされていない他のトランザクションが後でコミットされるときに書き込み競合を検出できます。 TiDB が[悲観的トランザクションモード](/pessimistic-transaction.md)を使用する場合、動作は基本的に他のデータベースと同じです。詳細は[MySQL InnoDBとの違い](/pessimistic-transaction.md#difference-with-mysql-innodb)を参照してください。 TiDB は`FOR UPDATE`の`NOWAIT`修飾子をサポートしています。詳細は[TiDB ペシミスティックトランザクションモード](/pessimistic-transaction.md#behaviors)参照してください。 |
| `LOCK IN SHARE MODE`           | 互換性を保証するために、TiDB はこれら 3 つの修飾子を解析しますが、無視します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `TABLESAMPLE`                  | テーブルから行のサンプルを取得します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

## 例 {#examples}

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
Query OK, 0 rows affected (0.11 sec)

mysql> INSERT INTO t1 (c1) VALUES (1),(2),(3),(4),(5);
Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t1;
+----+----+
| id | c1 |
+----+----+
|  1 |  1 |
|  2 |  2 |
|  3 |  3 |
|  4 |  4 |
|  5 |  5 |
+----+----+
5 rows in set (0.00 sec)
```

```sql
mysql> SELECT AVG(s_quantity), COUNT(s_quantity) FROM stock TABLESAMPLE REGIONS();
+-----------------+-------------------+
| AVG(s_quantity) | COUNT(s_quantity) |
+-----------------+-------------------+
|         59.5000 |                 4 |
+-----------------+-------------------+
1 row in set (0.00 sec)

mysql> SELECT AVG(s_quantity), COUNT(s_quantity) FROM stock;
+-----------------+-------------------+
| AVG(s_quantity) | COUNT(s_quantity) |
+-----------------+-------------------+
|         54.9729 |           1000000 |
+-----------------+-------------------+
1 row in set (0.52 sec)
```

上の例では、 `tiup bench tpcc prepare`で生成されたデータを使用しています。最初のクエリは`TABLESAMPLE`の使用を示しています。

## MySQLの互換性 {#mysql-compatibility}

-   構文`SELECT ... INTO @variable`はサポートされていません。
-   構文`SELECT ... GROUP BY ... WITH ROLLUP`はサポートされていません。
-   MySQL 5.7のように、構文`SELECT .. GROUP BY expr` `GROUP BY expr ORDER BY expr`を意味しません。 TiDB は MySQL 8.0 の動作に一致し、デフォルトの順序を意味しません。
-   構文`SELECT ... TABLESAMPLE ...`は TiDB 拡張機能であり、MySQL ではサポートされていません。

## こちらも参照 {#see-also}

-   [入れる](/sql-statements/sql-statement-insert.md)
-   [消去](/sql-statements/sql-statement-delete.md)
-   [アップデート](/sql-statements/sql-statement-update.md)
-   [交換](/sql-statements/sql-statement-replace.md)
