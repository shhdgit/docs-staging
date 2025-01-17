---
title: SHOW GRANTS | TiDB SQL Statement Reference
summary: An overview of the usage of SHOW GRANTS for the TiDB database.
---

# 助成金を表示する {#show-grants}

このステートメントは、ユーザーに関連付けられた権限のリストを示します。 MySQL と同様、 `USAGE`権限はTiDB にログインできることを示します。

## あらすじ {#synopsis}

**ShowGrantsStmt:**

![ShowGrantsStmt](https://download.pingcap.com/images/docs/sqlgram/ShowGrantsStmt.png)

**ユーザー名:**

![Username](https://download.pingcap.com/images/docs/sqlgram/Username.png)

**ロールの使用:**

![UsingRoles](https://download.pingcap.com/images/docs/sqlgram/UsingRoles.png)

**役割名リスト:**

![RolenameList](https://download.pingcap.com/images/docs/sqlgram/RolenameList.png)

**役割名:**

![Rolename](https://download.pingcap.com/images/docs/sqlgram/Rolename.png)

## 例 {#examples}

```sql
mysql> SHOW GRANTS;
+-------------------------------------------+
| Grants for User                           |
+-------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' |
+-------------------------------------------+
1 row in set (0.00 sec)

mysql> SHOW GRANTS FOR 'u1';
ERROR 1141 (42000): There is no such grant defined for user 'u1' on host '%'
mysql> CREATE USER u1;
Query OK, 1 row affected (0.04 sec)

mysql> GRANT SELECT ON test.* TO u1;
Query OK, 0 rows affected (0.04 sec)

mysql> SHOW GRANTS FOR u1;
+------------------------------------+
| Grants for u1@%                    |
+------------------------------------+
| GRANT USAGE ON *.* TO 'u1'@'%'     |
| GRANT Select ON test.* TO 'u1'@'%' |
+------------------------------------+
2 rows in set (0.00 sec)
```

## MySQLの互換性 {#mysql-compatibility}

TiDB の`SHOW GRANTS`ステートメントは MySQL と完全な互換性があります。互換性の違いを見つけた場合は、 [GitHub の問題](https://github.com/pingcap/tidb/issues/new/choose)を介して報告してください。

## こちらも参照 {#see-also}

-   [ユーザーの作成を表示](/sql-statements/sql-statement-show-create-user.md)
-   [許す](/sql-statements/sql-statement-grant-privileges.md)
