---
title: Timeouts in TiDB
summary: Learn about timeouts in TiDB, and solutions for troubleshooting errors.
---

# TiDB のタイムアウト {#timeouts-in-tidb}

このドキュメントでは、エラーのトラブルシューティングに役立つ TiDB のさまざまなタイムアウトについて説明します。

## GC タイムアウト {#gc-timeout}

TiDB のトランザクション実装では、MVCC (Multiple Version Concurrency Control) メカニズムが使用されます。新しく書き込まれたデータが古いデータを上書きする場合、古いデータは置き換えられず、新しく書き込まれたデータと一緒に保持されます。バージョンはタイムスタンプによって区別されます。 TiDB は、定期的なガベージ コレクション (GC) のメカニズムを使用して、不要になった古いデータをクリーンアップします。

デフォルトでは、各 MVCC バージョン (整合性スナップショット) は 10 分間保持されます。読み取りに 10 分以上かかるトランザクションは、エラー`GC life time is shorter than transaction duration`を受け取ります。

たとえば、フル バックアップに**Mydumper を**使用している場合 ( **Mydumper は**一貫性のあるスナップショットをバックアップする)、より長い読み取り時間が必要な場合は、TiDB の`mysql.tidb`テーブルの値`tikv_gc_life_time`を調整して、MVCC バージョンの保持時間を増やすことができます。 `tikv_gc_life_time`グローバルかつ即座に有効になることに注意してください。値を増やすと、既存のすべてのスナップショットの存続時間が長くなり、値を減らすと、すべてのスナップショットの存続時間がすぐに短くなります。 MVCC のバージョンが多すぎると、TiKV の処理効率に影響します。したがって、 **Mydumper**で完全バックアップを実行した後、時間内に`tikv_gc_life_time`前の設定に戻す必要があります。

GC の詳細については、 [GCの概要](/garbage-collection-overview.md)を参照してください。

## トランザクションタイムアウト {#transaction-timeout}

GC は進行中のトランザクションには影響しません。ただし、実行できる悲観的トランザクションの数には依然として上限があり、トランザクション タイムアウトの制限とトランザクションで使用されるメモリの制限があります。 TiDB プロファイルの`[performance]`カテゴリでトランザクション タイムアウトを`max-txn-ttl`ずつ変更できます (デフォルトでは`60`分)。

`INSERT INTO t10 SELECT * FROM t1`などの SQL ステートメントは GC の影響を受けませんが、 `max-txn-ttl`超えるとタイムアウトによりロールバックされます。

## SQL実行タイムアウト {#sql-execution-timeout}

TiDB は、単一の SQL ステートメント`0` `max_execution_time`も提供します。 `max_execution_time`現在、 `SELECT`ステートメントだけでなく、すべてのタイプのステートメントに対して有効です。単位は`ms`ですが、実際の精度はミリ秒レベルではなく`100ms`レベルです。

## JDBCクエリのタイムアウト {#jdbc-query-timeout}

MySQL JDBC のクエリ タイムアウト設定`setQueryTimeout()`は、TiDB では機能し***ませ***ん。これは、クライアントがタイムアウトを検出すると、データベースに`KILL`コマンドを送信するためです。ただし、 tidb サーバーは負荷分散されており、間違った tidb サーバーでの接続が終了するのを避けるために`KILL`コマンドは実行されません。クエリのタイムアウト効果を確認するには、 `MAX_EXECUTION_TIME`を使用する必要があります。

TiDB は、次の MySQL 互換のタイムアウト制御パラメータを提供します。

-   **wait_timeout**は、 Javaアプリケーションへの接続の非対話型アイドル タイムアウトを制御します。 TiDB v5.4 以降、デフォルト値`wait_timeout`は`28800`秒、つまり 8 時間です。 v5.4 より前の TiDB バージョンの場合、デフォルト値は`0`で、タイムアウトが無制限であることを意味します。
-   **interactive_timeout**は、 Javaアプリケーションへの接続の対話型アイドル タイムアウトを制御します。デフォルトの値は`8 hours`です。
-   **max_execution_time**は、接続における SQL 実行のタイムアウトを制御します。デフォルトの値は`0`で、接続は無限にビジー状態になります。つまり、SQL ステートメントが無限に長時間実行されます。

ただし、実際の本番環境では、アイドル状態の接続や無制限に実行される SQL ステートメントは、データベースとアプリケーションの両方に悪影響を及ぼします。アプリケーションの接続文字列でこれら 2 つのセッション レベルの変数を構成すると、アイドル接続と SQL ステートメントの無期限の実行を回避できます。たとえば、次のように設定します。

-   `sessionVariables=wait_timeout=3600` （1時間）
-   `sessionVariables=max_execution_time=300000` （5分）