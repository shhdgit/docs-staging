---
title: SHOW GRANTS | TiDB SQL Statement Reference
summary: An overview of the usage of SHOW GRANTS for the TiDB database.
aliases: ['/docs/dev/sql-statements/sql-statement-show-grants/','/docs/dev/reference/sql/statements/show-grants/']
---

# SHOW GRANTS

This statement shows a list of privileges associated with a user. As in MySQL, the `USAGE` privileges denotes the ability to login to TiDB.

## Synopsis

**ShowGrantsStmt:**

![ShowGrantsStmt](https://download.pingcap.com/images/docs/sqlgram/ShowGrantsStmt.png)

**Username:**

![Username](https://download.pingcap.com/images/docs/sqlgram/Username.png)

**UsingRoles:**

![UsingRoles](https://download.pingcap.com/images/docs/sqlgram/UsingRoles.png)

**RolenameList:**

![RolenameList](https://download.pingcap.com/images/docs/sqlgram/RolenameList.png)

**Rolename:**

![Rolename](https://download.pingcap.com/images/docs/sqlgram/Rolename.png)

## Examples

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

## MySQL compatibility

The `SHOW GRANTS` statement in TiDB is fully compatible with MySQL. If you find any compatibility differences, report them via [an issue on GitHub](https://github.com/pingcap/tidb/issues/new/choose).

## See also

* [SHOW CREATE USER](/sql-statements/sql-statement-show-create-user.md)
* [GRANT](/sql-statements/sql-statement-grant-privileges.md)
