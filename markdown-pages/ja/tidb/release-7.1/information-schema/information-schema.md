---
title: Information Schema
summary: TiDB implements the ANSI-standard information_schema for viewing system metadata.
---

# 情報スキーマ {#information-schema}

情報スキーマは、システム メタデータを表示する ANSI 標準の方法を提供します。 TiDB は、MySQL との互換性のために含まれるテーブルに加えて、多数のカスタム`INFORMATION_SCHEMA`テーブルも提供します。

多くの`INFORMATION_SCHEMA`テーブルには、対応する`SHOW`コマンドがあります。クエリ`INFORMATION_SCHEMA`の利点は、テーブル間を結合できることです。

## MySQL の互換性に関するテーブル {#tables-for-mysql-compatibility}

<CustomContent platform="tidb">

| テーブル名                                                                                                                      | 説明                                                                              |
| -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| [`CHARACTER_SETS`](/information-schema/information-schema-character-sets.md)                                               | サーバーがサポートする文字セットのリストを提供します。                                                     |
| [`COLLATIONS`](/information-schema/information-schema-collations.md)                                                       | サーバーがサポートする照合順序のリストを提供します。                                                      |
| [`COLLATION_CHARACTER_SET_APPLICABILITY`](/information-schema/information-schema-collation-character-set-applicability.md) | どの照合順序がどの文字セットに適用されるかを説明します。                                                    |
| [`COLUMNS`](/information-schema/information-schema-columns.md)                                                             | すべてのテーブルの列のリストを提供します。                                                           |
| `COLUMN_PRIVILEGES`                                                                                                        | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `COLUMN_STATISTICS`                                                                                                        | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`ENGINES`](/information-schema/information-schema-engines.md)                                                             | サポートされているstorageエンジンのリストを提供します。                                                 |
| `EVENTS`                                                                                                                   | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `FILES`                                                                                                                    | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `GLOBAL_STATUS`                                                                                                            | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `GLOBAL_VARIABLES`                                                                                                         | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`KEY_COLUMN_USAGE`](/information-schema/information-schema-key-column-usage.md)                                           | 主キー制約など、列のキー制約について説明します。                                                        |
| `OPTIMIZER_TRACE`                                                                                                          | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `PARAMETERS`                                                                                                               | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`PARTITIONS`](/information-schema/information-schema-partitions.md)                                                       | テーブルパーティションのリストを提供します。                                                          |
| `PLUGINS`                                                                                                                  | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`PROCESSLIST`](/information-schema/information-schema-processlist.md)                                                     | コマンド`SHOW PROCESSLIST`と同様の情報を提供します。                                             |
| `PROFILING`                                                                                                                | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `REFERENTIAL_CONSTRAINTS`                                                                                                  | `FOREIGN KEY`制約に関する情報を提供します。                                                    |
| `ROUTINES`                                                                                                                 | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`SCHEMATA`](/information-schema/information-schema-schemata.md)                                                           | `SHOW DATABASES`と同様の情報を提供します。                                                   |
| `SCHEMA_PRIVILEGES`                                                                                                        | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `SESSION_STATUS`                                                                                                           | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`SESSION_VARIABLES`](/information-schema/information-schema-session-variables.md)                                         | コマンド`SHOW SESSION VARIABLES`と同様の機能を提供します。                                       |
| [`STATISTICS`](/information-schema/information-schema-statistics.md)                                                       | テーブルインデックスに関する情報を提供します。                                                         |
| [`TABLES`](/information-schema/information-schema-tables.md)                                                               | 現在のユーザーが表示できるテーブルのリストを提供します。 `SHOW TABLES`と同様です。                                |
| `TABLESPACES`                                                                                                              | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`TABLE_CONSTRAINTS`](/information-schema/information-schema-table-constraints.md)                                         | 主キー、一意のインデックス、外部キーに関する情報を提供します。                                                 |
| `TABLE_PRIVILEGES`                                                                                                         | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `TRIGGERS`                                                                                                                 | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`USER_ATTRIBUTES`](/information-schema/information-schema-user-attributes.md)                                             | ユーザーのコメントやユーザー属性に関する情報をまとめます。                                                   |
| [`USER_PRIVILEGES`](/information-schema/information-schema-user-privileges.md)                                             | 現在のユーザーに関連付けられている権限を要約します。                                                      |
| [`VARIABLES_INFO`](/information-schema/information-schema-variables-info.md)                                               | TiDB システム変数に関する情報を提供します。                                                        |
| [`VIEWS`](/information-schema/information-schema-views.md)                                                                 | 現在のユーザーが表示できるビューのリストを提供します。ランニング`SHOW FULL TABLES WHERE table_type = 'VIEW'`と同様 |

</CustomContent>

<CustomContent platform="tidb-cloud">

| テーブル名                                                                                                                      | 説明                                                                              |
| -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| [`CHARACTER_SETS`](/information-schema/information-schema-character-sets.md)                                               | サーバーがサポートする文字セットのリストを提供します。                                                     |
| [`COLLATIONS`](/information-schema/information-schema-collations.md)                                                       | サーバーがサポートする照合順序のリストを提供します。                                                      |
| [`COLLATION_CHARACTER_SET_APPLICABILITY`](/information-schema/information-schema-collation-character-set-applicability.md) | どの照合順序がどの文字セットに適用されるかを説明します。                                                    |
| [`COLUMNS`](/information-schema/information-schema-columns.md)                                                             | すべてのテーブルの列のリストを提供します。                                                           |
| `COLUMN_PRIVILEGES`                                                                                                        | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `COLUMN_STATISTICS`                                                                                                        | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`ENGINES`](/information-schema/information-schema-engines.md)                                                             | サポートされているstorageエンジンのリストを提供します。                                                 |
| `EVENTS`                                                                                                                   | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `FILES`                                                                                                                    | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `GLOBAL_STATUS`                                                                                                            | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `GLOBAL_VARIABLES`                                                                                                         | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`KEY_COLUMN_USAGE`](/information-schema/information-schema-key-column-usage.md)                                           | 主キー制約など、列のキー制約について説明します。                                                        |
| `OPTIMIZER_TRACE`                                                                                                          | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `PARAMETERS`                                                                                                               | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`PARTITIONS`](/information-schema/information-schema-partitions.md)                                                       | テーブルパーティションのリストを提供します。                                                          |
| `PLUGINS`                                                                                                                  | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`PROCESSLIST`](/information-schema/information-schema-processlist.md)                                                     | コマンド`SHOW PROCESSLIST`と同様の情報を提供します。                                             |
| `PROFILING`                                                                                                                | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `REFERENTIAL_CONSTRAINTS`                                                                                                  | `FOREIGN KEY`制約に関する情報を提供します。                                                    |
| `ROUTINES`                                                                                                                 | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`SCHEMATA`](/information-schema/information-schema-schemata.md)                                                           | `SHOW DATABASES`と同様の情報を提供します。                                                   |
| `SCHEMA_PRIVILEGES`                                                                                                        | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `SESSION_STATUS`                                                                                                           | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`SESSION_VARIABLES`](/information-schema/information-schema-session-variables.md)                                         | コマンド`SHOW SESSION VARIABLES`と同様の機能を提供します。                                       |
| [`STATISTICS`](/information-schema/information-schema-statistics.md)                                                       | テーブルインデックスに関する情報を提供します。                                                         |
| [`TABLES`](/information-schema/information-schema-tables.md)                                                               | 現在のユーザーが表示できるテーブルのリストを提供します。 `SHOW TABLES`と同様です。                                |
| `TABLESPACES`                                                                                                              | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`TABLE_CONSTRAINTS`](/information-schema/information-schema-table-constraints.md)                                         | 主キー、一意のインデックス、外部キーに関する情報を提供します。                                                 |
| `TABLE_PRIVILEGES`                                                                                                         | TiDB では実装されていません。ゼロ行を返します。                                                      |
| `TRIGGERS`                                                                                                                 | TiDB では実装されていません。ゼロ行を返します。                                                      |
| [`USER_ATTRIBUTES`](/information-schema/information-schema-user-attributes.md)                                             | ユーザーのコメントやユーザー属性に関する情報をまとめます。                                                   |
| [`USER_PRIVILEGES`](/information-schema/information-schema-user-privileges.md)                                             | 現在のユーザーに関連付けられている権限を要約します。                                                      |
| [`VARIABLES_INFO`](/information-schema/information-schema-variables-info.md)                                               | TiDB システム変数に関する情報を提供します。                                                        |
| [`VIEWS`](/information-schema/information-schema-views.md)                                                                 | 現在のユーザーが表示できるビューのリストを提供します。ランニング`SHOW FULL TABLES WHERE table_type = 'VIEW'`と同様 |

</CustomContent>

## TiDB 拡張機能であるテーブル {#tables-that-are-tidb-extensions}

<CustomContent platform="tidb">

| テーブル名                                                                                            | 説明                                                    |
| ------------------------------------------------------------------------------------------------ | ----------------------------------------------------- |
| [`ANALYZE_STATUS`](/information-schema/information-schema-analyze-status.md)                     | 統計を収集するタスクに関する情報を提供します。                               |
| [`CLIENT_ERRORS_SUMMARY_BY_HOST`](/information-schema/client-errors-summary-by-host.md)          | クライアント要求によって生成され、クライアントに返されたエラーと警告の概要を提供します。          |
| [`CLIENT_ERRORS_SUMMARY_BY_USER`](/information-schema/client-errors-summary-by-user.md)          | クライアントによって生成されたエラーと警告の概要を提供します。                       |
| [`CLIENT_ERRORS_SUMMARY_GLOBAL`](/information-schema/client-errors-summary-global.md)            | クライアントによって生成されたエラーと警告の概要を提供します。                       |
| [`CLUSTER_CONFIG`](/information-schema/information-schema-cluster-config.md)                     | TiDB クラスター全体の構成設定に関する詳細を提供します。                        |
| `CLUSTER_DEADLOCKS`                                                                              | `DEADLOCKS`テーブルのクラスター レベルのビューを提供します。                  |
| [`CLUSTER_HARDWARE`](/information-schema/information-schema-cluster-hardware.md)                 | 各 TiDBコンポーネントで検出された基礎となる物理ハードウェアの詳細を提供します。            |
| [`CLUSTER_INFO`](/information-schema/information-schema-cluster-info.md)                         | 現在のクラスター トポロジの詳細を提供します。                               |
| [`CLUSTER_LOAD`](/information-schema/information-schema-cluster-load.md)                         | クラスター内の TiDB サーバーの現在の負荷情報を提供します。                      |
| [`CLUSTER_LOG`](/information-schema/information-schema-cluster-log.md)                           | TiDB クラスター全体のログを提供します。                                |
| `CLUSTER_MEMORY_USAGE`                                                                           | `MEMORY_USAGE`テーブルのクラスター レベルのビューを提供します。               |
| `CLUSTER_MEMORY_USAGE_OPS_HISTORY`                                                               | `MEMORY_USAGE_OPS_HISTORY`テーブルのクラスター レベルのビューを提供します。   |
| `CLUSTER_PROCESSLIST`                                                                            | `PROCESSLIST`テーブルのクラスター レベルのビューを提供します。                |
| `CLUSTER_SLOW_QUERY`                                                                             | `SLOW_QUERY`テーブルのクラスター レベルのビューを提供します。                 |
| `CLUSTER_STATEMENTS_SUMMARY`                                                                     | `STATEMENTS_SUMMARY`テーブルのクラスター レベルのビューを提供します。         |
| `CLUSTER_STATEMENTS_SUMMARY_HISTORY`                                                             | `STATEMENTS_SUMMARY_HISTORY`テーブルのクラスター レベルのビューを提供します。 |
| `CLUSTER_TIDB_TRX`                                                                               | `TIDB_TRX`テーブルのクラスター レベルのビューを提供します。                   |
| [`CLUSTER_SYSTEMINFO`](/information-schema/information-schema-cluster-systeminfo.md)             | クラスター内のサーバーのカーネル パラメーター構成の詳細を提供します。                   |
| [`DATA_LOCK_WAITS`](/information-schema/information-schema-data-lock-waits.md)                   | TiKVサーバー上のロック待機情報を提供します。                              |
| [`DDL_JOBS`](/information-schema/information-schema-ddl-jobs.md)                                 | `ADMIN SHOW DDL JOBS`と同様の出力を提供します                     |
| [`DEADLOCKS`](/information-schema/information-schema-deadlocks.md)                               | 最近発生したいくつかのデッドロック エラーの情報を提供します。                       |
| [`INSPECTION_RESULT`](/information-schema/information-schema-inspection-result.md)               | 内部診断チェックをトリガーします。                                     |
| [`INSPECTION_RULES`](/information-schema/information-schema-inspection-rules.md)                 | 実行された内部診断チェックのリスト。                                    |
| [`INSPECTION_SUMMARY`](/information-schema/information-schema-inspection-summary.md)             | 重要な監視指標の要約レポート。                                       |
| [`MEMORY_USAGE`](/information-schema/information-schema-memory-usage.md)                         | 現在の TiDB インスタンスのメモリ使用量。                               |
| [`MEMORY_USAGE_OPS_HISTORY`](/information-schema/information-schema-memory-usage-ops-history.md) | メモリ関連の操作の履歴と現在の TiDB インスタンスの実行基盤。                     |
| [`METRICS_SUMMARY`](/information-schema/information-schema-metrics-summary.md)                   | Prometheus から抽出されたメトリクスの概要。                           |
| `METRICS_SUMMARY_BY_LABEL`                                                                       | `METRICS_SUMMARY`表を参照してください。                          |
| [`METRICS_TABLES`](/information-schema/information-schema-metrics-tables.md)                     | `METRICS_SCHEMA`のテーブルの PromQL 定義を提供します。               |
| [`PLACEMENT_POLICIES`](/information-schema/information-schema-placement-policies.md)             | すべての配置ポリシーに関する情報を提供します。                               |
| [`SEQUENCES`](/information-schema/information-schema-sequences.md)                               | シーケンスの TiDB 実装は MariaDB に基づいています。                     |
| [`SLOW_QUERY`](/information-schema/information-schema-slow-query.md)                             | 現在の TiDBサーバー上の遅いクエリに関する情報を提供します。                      |
| [`STATEMENTS_SUMMARY`](/statement-summary-tables.md)                                             | MySQL の Performance_schema ステートメントの概要に似ています。          |
| [`STATEMENTS_SUMMARY_HISTORY`](/statement-summary-tables.md)                                     | MySQL の Performance_schema ステートメントの概要履歴に似ています。        |
| [`TABLE_STORAGE_STATS`](/information-schema/information-schema-table-storage-stats.md)           | storage内のテーブル サイズの詳細を提供します。                           |
| [`TIDB_HOT_REGIONS`](/information-schema/information-schema-tidb-hot-regions.md)                 | どの地域がホットであるかに関する統計を提供します。                             |
| [`TIDB_HOT_REGIONS_HISTORY`](/information-schema/information-schema-tidb-hot-regions-history.md) | どのリージョンがホットであるかに関する履歴統計を提供します。                        |
| [`TIDB_INDEXES`](/information-schema/information-schema-tidb-indexes.md)                         | TiDB テーブルに関するインデックス情報を提供します。                          |
| [`TIDB_SERVERS_INFO`](/information-schema/information-schema-tidb-servers-info.md)               | TiDB サーバー (つまり、 tidb-serverコンポーネント) のリストを提供します。       |
| [`TIDB_TRX`](/information-schema/information-schema-tidb-trx.md)                                 | TiDB ノード上で実行されているトランザクションの情報を提供します。                   |
| [`TIFLASH_REPLICA`](/information-schema/information-schema-tiflash-replica.md)                   | TiFlashレプリカの詳細を提供します。                                 |
| [`TIKV_REGION_PEERS`](/information-schema/information-schema-tikv-region-peers.md)               | リージョンが保存される場所に関する詳細を提供します。                            |
| [`TIKV_REGION_STATUS`](/information-schema/information-schema-tikv-region-status.md)             | 地域に関する統計を提供します。                                       |
| [`TIKV_STORE_STATUS`](/information-schema/information-schema-tikv-store-status.md)               | TiKV サーバーに関する基本情報を提供します。                              |

</CustomContent>

<CustomContent platform="tidb-cloud">

| テーブル名                                                                                              | 説明                                                                          |
| -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [`ANALYZE_STATUS`](/information-schema/information-schema-analyze-status.md)                       | 統計を収集するタスクに関する情報を提供します。                                                     |
| [`CLIENT_ERRORS_SUMMARY_BY_HOST`](/information-schema/client-errors-summary-by-host.md)            | クライアント要求によって生成され、クライアントに返されたエラーと警告の概要を提供します。                                |
| [`CLIENT_ERRORS_SUMMARY_BY_USER`](/information-schema/client-errors-summary-by-user.md)            | クライアントによって生成されたエラーと警告の概要を提供します。                                             |
| [`CLIENT_ERRORS_SUMMARY_GLOBAL`](/information-schema/client-errors-summary-global.md)              | クライアントによって生成されたエラーと警告の概要を提供します。                                             |
| [`CLUSTER_CONFIG`](https://docs.pingcap.com/tidb/stable/information-schema-cluster-config)         | TiDB クラスター全体の構成設定に関する詳細を提供します。この表はTiDB Cloudには適用されません。                      |
| `CLUSTER_DEADLOCKS`                                                                                | `DEADLOCKS`テーブルのクラスター レベルのビューを提供します。                                        |
| [`CLUSTER_HARDWARE`](https://docs.pingcap.com/tidb/stable/information-schema-cluster-hardware)     | 各 TiDBコンポーネントで検出された基礎となる物理ハードウェアの詳細を提供します。この表はTiDB Cloudには適用されません。          |
| [`CLUSTER_INFO`](/information-schema/information-schema-cluster-info.md)                           | 現在のクラスター トポロジの詳細を提供します。                                                     |
| [`CLUSTER_LOAD`](https://docs.pingcap.com/tidb/stable/information-schema-cluster-load)             | クラスター内の TiDB サーバーの現在の負荷情報を提供します。この表はTiDB Cloudには適用されません。                    |
| [`CLUSTER_LOG`](https://docs.pingcap.com/tidb/stable/information-schema-cluster-log)               | TiDB クラスター全体のログを提供します。この表はTiDB Cloudには適用されません。                              |
| `CLUSTER_MEMORY_USAGE`                                                                             | `MEMORY_USAGE`テーブルのクラスター レベルのビューを提供します。この表はTiDB Cloudには適用されません。             |
| `CLUSTER_MEMORY_USAGE_OPS_HISTORY`                                                                 | `MEMORY_USAGE_OPS_HISTORY`テーブルのクラスター レベルのビューを提供します。この表はTiDB Cloudには適用されません。 |
| `CLUSTER_PROCESSLIST`                                                                              | `PROCESSLIST`テーブルのクラスター レベルのビューを提供します。                                      |
| `CLUSTER_SLOW_QUERY`                                                                               | `SLOW_QUERY`テーブルのクラスター レベルのビューを提供します。                                       |
| `CLUSTER_STATEMENTS_SUMMARY`                                                                       | `STATEMENTS_SUMMARY`テーブルのクラスター レベルのビューを提供します。                               |
| `CLUSTER_STATEMENTS_SUMMARY_HISTORY`                                                               | `STATEMENTS_SUMMARY_HISTORY`テーブルのクラスター レベルのビューを提供します。                       |
| `CLUSTER_TIDB_TRX`                                                                                 | `TIDB_TRX`テーブルのクラスター レベルのビューを提供します。                                         |
| [`CLUSTER_SYSTEMINFO`](https://docs.pingcap.com/tidb/stable/information-schema-cluster-systeminfo) | クラスター内のサーバーのカーネル パラメーター構成の詳細を提供します。この表はTiDB Cloudには適用されません。                 |
| [`DATA_LOCK_WAITS`](/information-schema/information-schema-data-lock-waits.md)                     | TiKVサーバー上のロック待機情報を提供します。                                                    |
| [`DDL_JOBS`](/information-schema/information-schema-ddl-jobs.md)                                   | `ADMIN SHOW DDL JOBS`と同様の出力を提供します                                           |
| [`DEADLOCKS`](/information-schema/information-schema-deadlocks.md)                                 | 最近発生したいくつかのデッドロック エラーの情報を提供します。                                             |
| [`INSPECTION_RESULT`](https://docs.pingcap.com/tidb/stable/information-schema-inspection-result)   | 内部診断チェックをトリガーします。この表はTiDB Cloudには適用されません。                                   |
| [`INSPECTION_RULES`](https://docs.pingcap.com/tidb/stable/information-schema-inspection-rules)     | 実行された内部診断チェックのリスト。この表はTiDB Cloudには適用されません。                                  |
| [`INSPECTION_SUMMARY`](https://docs.pingcap.com/tidb/stable/information-schema-inspection-summary) | 重要な監視指標の要約レポート。この表はTiDB Cloudには適用されません。                                     |
| [`MEMORY_USAGE`](/information-schema/information-schema-memory-usage.md)                           | 現在の TiDB インスタンスのメモリ使用量。                                                     |
| [`MEMORY_USAGE_OPS_HISTORY`](/information-schema/information-schema-memory-usage-ops-history.md)   | メモリ関連の操作の履歴と現在の TiDB インスタンスの実行基盤。                                           |
| [`METRICS_SUMMARY`](https://docs.pingcap.com/tidb/stable/information-schema-metrics-summary)       | Prometheus から抽出されたメトリクスの概要。この表はTiDB Cloudには適用されません。                         |
| `METRICS_SUMMARY_BY_LABEL`                                                                         | `METRICS_SUMMARY`表を参照してください。                                                |
| [`METRICS_TABLES`](https://docs.pingcap.com/tidb/stable/information-schema-metrics-tables)         | `METRICS_SCHEMA`のテーブルの PromQL 定義を提供します。この表はTiDB Cloudには適用されません。             |
| [`PLACEMENT_POLICIES`](https://docs.pingcap.com/tidb/stable/information-schema-placement-policies) | すべての配置ポリシーに関する情報を提供します。この表はTiDB Cloudには適用されません。                             |
| [`SEQUENCES`](/information-schema/information-schema-sequences.md)                                 | シーケンスの TiDB 実装は MariaDB に基づいています。                                           |
| [`SLOW_QUERY`](/information-schema/information-schema-slow-query.md)                               | 現在の TiDBサーバー上の遅いクエリに関する情報を提供します。                                            |
| [`STATEMENTS_SUMMARY`](/statement-summary-tables.md)                                               | MySQL の Performance_schema ステートメントの概要に似ています。                                |
| [`STATEMENTS_SUMMARY_HISTORY`](/statement-summary-tables.md)                                       | MySQL の Performance_schema ステートメントの概要履歴に似ています。                              |
| [`TABLE_STORAGE_STATS`](/information-schema/information-schema-table-storage-stats.md)             | storage内のテーブル サイズの詳細を提供します。                                                 |
| [`TIDB_HOT_REGIONS`](https://docs.pingcap.com/tidb/stable/information-schema-tidb-hot-regions)     | どの地域がホットであるかに関する統計を提供します。この表はTiDB Cloudには適用されません。                           |
| [`TIDB_HOT_REGIONS_HISTORY`](/information-schema/information-schema-tidb-hot-regions-history.md)   | どのリージョンがホットであるかに関する履歴統計を提供します。                                              |
| [`TIDB_INDEXES`](/information-schema/information-schema-tidb-indexes.md)                           | TiDB テーブルに関するインデックス情報を提供します。                                                |
| [`TIDB_SERVERS_INFO`](/information-schema/information-schema-tidb-servers-info.md)                 | TiDB サーバー (つまり、 tidb-serverコンポーネント) のリストを提供します。                             |
| [`TIDB_TRX`](/information-schema/information-schema-tidb-trx.md)                                   | TiDB ノード上で実行されているトランザクションの情報を提供します。                                         |
| [`TIFLASH_REPLICA`](/information-schema/information-schema-tiflash-replica.md)                     | TiFlashレプリカの詳細を提供します。                                                       |
| [`TIKV_REGION_PEERS`](/information-schema/information-schema-tikv-region-peers.md)                 | リージョンが保存される場所に関する詳細を提供します。                                                  |
| [`TIKV_REGION_STATUS`](/information-schema/information-schema-tikv-region-status.md)               | 地域に関する統計を提供します。                                                             |
| [`TIKV_STORE_STATUS`](/information-schema/information-schema-tikv-store-status.md)                 | TiKV サーバーに関する基本情報を提供します。                                                    |

</CustomContent>
