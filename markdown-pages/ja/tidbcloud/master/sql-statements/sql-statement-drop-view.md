---
title: DROP VIEW | TiDB SQL Statement Reference
summary: An overview of the usage of DROP VIEW for the TiDB database.
---

# ドロップビュー {#drop-view}

この文は、現在選択されているデータベースからビュー・オブジェクトを削除します。ビューが参照する実表には影響しません。

## あらすじ {#synopsis}

```ebnf+diagram
DropViewStmt ::=
    'DROP' 'VIEW' ( 'IF' 'EXISTS' )? TableNameList RestrictOrCascadeOpt

TableNameList ::=
    TableName ( ',' TableName )*

TableName ::=
    Identifier ('.' Identifier)?
```

## 例 {#examples}

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
Query OK, 0 rows affected (0.11 sec)

mysql> INSERT INTO t1 (c1) VALUES (1),(2),(3),(4),(5);
Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> CREATE VIEW v1 AS SELECT * FROM t1 WHERE c1 > 2;
Query OK, 0 rows affected (0.11 sec)

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

mysql> SELECT * FROM v1;
+----+----+
| id | c1 |
+----+----+
|  3 |  3 |
|  4 |  4 |
|  5 |  5 |
+----+----+
3 rows in set (0.00 sec)

mysql> DROP VIEW v1;
Query OK, 0 rows affected (0.23 sec)

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

## MySQLの互換性 {#mysql-compatibility}

TiDB の`DROP VIEW`ステートメントは MySQL と完全な互換性があります。互換性の違いを見つけた場合は、 [GitHub の問題](https://github.com/pingcap/tidb/issues/new/choose)を介して報告してください。

## こちらも参照 {#see-also}

-   [ビューの作成](/sql-statements/sql-statement-create-view.md)
-   [ドロップテーブル](/sql-statements/sql-statement-drop-table.md)
