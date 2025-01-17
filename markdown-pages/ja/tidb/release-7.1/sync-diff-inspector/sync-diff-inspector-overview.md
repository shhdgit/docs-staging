---
title: sync-diff-inspector User Guide
summary: Use sync-diff-inspector to compare data and repair inconsistent data.
---

# sync-diff-inspector ユーザーガイド {#sync-diff-inspector-user-guide}

[同期差分インスペクター](https://github.com/pingcap/tidb-tools/tree/master/sync_diff_inspector)データベースに保存されているデータを MySQL プロトコルと比較するために使用されるツールです。たとえば、MySQL のデータと TiDB のデータ、MySQL のデータと MySQL のデータ、または TiDB のデータと TiDB のデータを比較できます。さらに、このツールを使用して、少量のデータに不整合があるシナリオでデータを修復することもできます。

このガイドでは、sync-diff-inspector の主要な機能を紹介し、このツールの構成方法と使用方法について説明します。 sync-diff-inspector をダウンロードするには、次のいずれかの方法を使用します。

-   バイナリパッケージ。 sync-diff-inspector バイナリ パッケージはTiDB Toolkitに含まれています。 TiDB Toolkitをダウンロードするには、 [TiDB ツールをダウンロード](/download-ecosystem-tools.md)を参照してください。
-   ドッカーイメージ。次のコマンドを実行してダウンロードします。

    
    ```shell
    docker pull pingcap/tidb-tools:latest
    ```

## 主な特徴 {#key-features}

-   テーブルのスキーマとデータを比較する
-   データの不整合が存在する場合にデータを修復するために使用される SQL ステートメントを生成します。
-   サポート[異なるスキーマ名またはテーブル名を持つテーブルのデータ チェック](/sync-diff-inspector/route-diff.md)
-   サポート[シャーディングシナリオでのデータチェック](/sync-diff-inspector/shard-diff.md)
-   サポート[TiDB アップストリーム/ダウンストリーム クラスターのデータ チェック](/sync-diff-inspector/upstream-downstream-diff.md)
-   サポート[DM レプリケーション シナリオでのデータ チェック](/sync-diff-inspector/dm-diff.md)

## sync-diff-inspector の制限事項 {#restrictions-of-sync-diff-inspector}

-   オンライン チェックは、MySQL と TiDB 間のデータ移行ではサポートされていません。上流-下流チェックリストにデータが書き込まれていないこと、および特定の範囲のデータが変更されていないことを確認してください。 `range`を設定すると、この範囲のデータを確認できます。

-   TiDB と MySQL では、 `FLOAT` 、 `DOUBLE` 、およびその他の浮動小数点型は異なる方法で実装されます。 `FLOAT`と`DOUBLE` 、チェックサムの計算にそれぞれ 6 桁と 15 桁の有効数字を取ります。この機能を使用したくない場合は、 `ignore-columns`を設定してこれらの列のチェックをスキップします。

-   主キーまたは一意のインデックスを含まないテーブルのチェックをサポートします。ただし、データに一貫性がない場合、生成された SQL ステートメントはデータを正しく修復できない可能性があります。

## sync-diff-inspector のデータベース権限 {#database-privileges-for-sync-diff-inspector}

sync-diff-inspector はテーブル スキーマの情報を取得し、データをクエリする必要があります。必要なデータベース権限は次のとおりです。

-   上流データベース
    -   `SELECT` (比較のためにデータをチェックします)
    -   `SHOW_DATABASES` (ビューデータベース名)
    -   `RELOAD` (テーブルスキーマを表示)
-   ダウンストリームデータベース
    -   `SELECT` (比較のためにデータをチェックします)
    -   `SHOW_DATABASES` (ビューデータベース名)
    -   `RELOAD` (テーブルスキーマを表示)

## コンフィグレーションファイルの説明 {#configuration-file-description}

sync-diff-inspector の構成は次の部分で構成されます。

-   `Global config` : チェックするスレッドの数、不整合なテーブルを修正するために SQL ステートメントをエクスポートするかどうか、データを比較するかどうか、上流または下流に存在しないテーブルのチェックをスキップするかどうかなどの一般的な設定。
-   `Databases config` : アップストリーム データベースとダウンストリーム データベースのインスタンスを構成します。
-   `Routes` : アップストリームの複数のスキーマ名がダウンストリームの単一スキーマ名と一致するためのルール**(オプション)** 。
-   `Task config` : チェック用のテーブルを設定します。一部のテーブルにアップストリーム データベースとダウンストリーム データベースの間に特定のマッピング関係がある場合、またはいくつかの特別な要件がある場合、これらのテーブルを構成する必要があります。
-   `Table config` : 無視される指定された範囲や列など、特定のテーブルの特別な構成**(オプション)** 。

以下は完全な構成ファイルの説明です。

-   注: 名前の後に`s`が付いている構成には複数の値を含めることができるため、構成値を含めるには角括弧`[]`を使用する必要があります。

```toml
# Diff Configuration.

######################### Global config #########################
# The number of goroutines created to check data. The number of connections between sync-diff-inspector and upstream/downstream databases is slightly greater than this value.
check-thread-count = 4

# If enabled, SQL statements is exported to fix inconsistent tables.
export-fix-sql = true

# Only compares the table structure instead of the data.
check-struct-only = false

# If enabled, sync-diff-inspector skips checking tables that do not exist in the upstream or downstream.
skip-non-existing-table = false

######################### Datasource config #########################
[data-sources]
[data-sources.mysql1] # mysql1 is the only custom ID for the database instance. It is used for the following `task.source-instances/task.target-instance` configuration.
    host = "127.0.0.1"
    port = 3306
    user = "root"
    password = ""  # The password for connecting to the upstream database. It can be plain text or Base64-encoded.

    # (optional) Use mapping rules to match multiple upstream sharded tables. Rule1 and rule2 are configured in the following Routes section.
    route-rules = ["rule1", "rule2"]

[data-sources.tidb0]
    host = "127.0.0.1"
    port = 4000
    user = "root"
    password = ""  # The password for connecting to the downstream database. It can be plain text or Base64-encoded.

    # (optional) Use TLS to connect TiDB.
    # security.ca-path = ".../ca.crt"
    # security.cert-path = ".../cert.crt"
    # security.key-path = ".../key.crt"

    # (optional) Use the snapshot feature. If enabled, historical data is used for comparison.
    # snapshot = "386902609362944000"
    # When "snapshot" is set to "auto", the last syncpoints generated by TiCDC in the upstream and downstream are used for comparison. For details, see <https://github.com/pingcap/tidb-tools/issues/663>.
    # snapshot = "auto"

########################### Routes ##############################
# To compare the data of a large number of tables with different schema names or table names, or check the data of multiple upstream sharded tables and downstream table family, use the table-rule to configure the mapping relationship. You can configure the mapping rule only for the schema or table. Also, you can configure the mapping rules for both the schema and the table.
[routes]
[routes.rule1] # rule1 is the only custom ID for the configuration. It is used for the above `data-sources.route-rules` configuration.
schema-pattern = "test_*"      # Matches the schema name of the data source. Supports the wildcards "*" and "?"
table-pattern = "t_*"          # Matches the table name of the data source. Supports the wildcards "*" and "?"
target-schema = "test"         # The name of the schema in the target database
target-table = "t"             # The name of the target table
[routes.rule2]
schema-pattern = "test2_*"      # Matches the schema name of the data source. Supports the wildcards "*" and "?"
table-pattern = "t2_*"          # Matches the table name of the data source. Supports the wildcards "*" and "?"
target-schema = "test2"         # The name of the schema in the target database
target-table = "t2"             # The name of the target table

######################### task config #########################
# Configures the tables of the target database that need to be compared.
[task]
    # output-dir saves the following information:
    # 1 sql: The SQL file to fix tables that is generated after error is detected. One chunk corresponds to one SQL file.
    # 2 log: sync-diff.log
    # 3 summary: summary.txt
    # 4 checkpoint: a dir
    output-dir = "./output"
    # The upstream database. The value is the unique ID declared by data-sources.
    source-instances = ["mysql1"]
    # The downstream database. The value is the unique ID declared by data-sources.
    target-instance = "tidb0"
    # The tables of downstream databases to be compared. Each table needs to contain the schema name and the table name, separated by '.'
    # Use "?" to match any character and "*" to match characters of any length.
    # For detailed match rules, refer to golang regexp pkg: https://github.com/google/re2/wiki/Syntax.
    target-check-tables = ["schema*.table*", "!c.*", "test2.t2"]
    # (optional) Extra configurations for some tables, Config1 is defined in the following table config example.
    target-configs = ["config1"]

######################### Table config #########################
# Special configurations for specific tables. The tables to be configured must be in `task.target-check-tables`.
[table-configs.config1] # config1  is the only custom ID for this configuration. It is used for the above `task.target-configs` configuration.
# The name of the target table, you can use regular expressions to match multiple tables, but one table is not allowed to be matched by multiple special configurations at the same time.
target-tables = ["schema*.test*", "test2.t2"]
# (optional) Specifies the range of the data to be checked
# It needs to comply with the syntax of the WHERE clause in SQL.
range = "age > 10 AND age < 20"
# (optional) Specifies the column used to divide data into chunks. If you do not configure it,
# sync-diff-inspector chooses an appropriate column (primary key, unique key, or a field with index).
index-fields = ["col1","col2"]
# (optional) Ignores checking some columns such as some types (json, bit, blob, etc.)
# that sync-diff-inspector does not currently support.
# The floating-point data type behaves differently in TiDB and MySQL. You can use
# `ignore-columns` to skip checking these columns.
ignore-columns = ["",""]
# (optional) Specifies the size of the chunk for dividing the table. If not specified, this configuration can be deleted or be set as 0.
chunk-size = 0
# (optional) Specifies the "collation" for the table. If not specified, this configuration can be deleted or be set as an empty string.
collation = ""
```

## sync-diff-inspector を実行する {#run-sync-diff-inspector}

次のコマンドを実行します。


```bash
./sync_diff_inspector --config=./config.toml
```

このコマンドは、 `output-dir` of `config.toml`のチェック レポート`summary.txt`とログ`sync_diff.log`を出力します。 `output-dir`では、 `config. toml`ファイルのハッシュ値で名前を付けたフォルダも生成されます。このフォルダには、ブレークポイントのチェックポイントノード情報と、データ不整合時に生成されるSQLファイルが格納されます。

### 進捗情報 {#progress-information}

sync-diff-inspector は実行時に進行状況情報を`stdout`に送信します。進捗情報には、テーブル構造の比較結果、テーブルデータの比較結果、プログレスバーが含まれます。

> **ノート：**
>
> 表示効果を確保するには、表示ウィンドウの幅を 80 文字以上に保ってください。

```progress
A total of 2 tables need to be compared

Comparing the table structure of ``sbtest`.`sbtest96`` ... equivalent
Comparing the table structure of ``sbtest`.`sbtest99`` ... equivalent
Comparing the table data of ``sbtest`.`sbtest96`` ... failure
Comparing the table data of ``sbtest`.`sbtest99`` ...
_____________________________________________________________________________
Progress [==========================================================>--] 98% 193/200
```

```progress
A total of 2 tables need to be compared

Comparing the table structure of ``sbtest`.`sbtest96`` ... equivalent
Comparing the table structure of ``sbtest`.`sbtest99`` ... equivalent
Comparing the table data of ``sbtest`.`sbtest96`` ... failure
Comparing the table data of ``sbtest`.`sbtest99`` ... failure
_____________________________________________________________________________
Progress [============================================================>] 100% 0/0
The data of `sbtest`.`sbtest99` is not equal
The data of `sbtest`.`sbtest96` is not equal

The rest of tables are all equal.

A total of 2 tables have been compared, 0 tables finished, 2 tables failed, 0 tables skipped.
The patch file has been generated in
        'output/fix-on-tidb2/'
You can view the comparison details through 'output/sync_diff.log'
```

### 出力ファイル {#output-file}

出力ファイルのディレクトリ構造は次のとおりです。

```
output/
|-- checkpoint # Saves the breakpoint information
| |-- bbfec8cc8d1f58a5800e63aa73e5 # Config hash. The placeholder file which identifies the configuration file corresponding to the output directory (output/)
│ |-- DO_NOT_EDIT_THIS_DIR
│ └-- sync_diff_checkpoints.pb # The breakpoint information
|
|-- fix-on-target # Saves SQL files to fix data inconsistency
| |-- xxx.sql
| |-- xxx.sql
| └-- xxx.sql
|
|-- summary.txt # Saves the summary of the check results
└-- sync_diff.log # Saves the output log information when sync-diff-inspector is running
```

### ログ {#log}

sync-diff-inspector のログは`${output}/sync_diff.log`に保存されますが、このうち`${output}` `config.toml`ファイルの`output-dir`の値です。

### 進捗 {#progress}

実行中の sync-diff-inspector は定期的 (10 秒ごと) にチェックポイントの進行状況を出力。チェックポイントは`${output}/checkpoint/sync_diff_checkpoints.pb`にあり、そのうちの`${output}`はファイル`config.toml`の`output-dir`の値です。

### 結果 {#result}

チェックが完了すると、sync-diff-inspector はレポートを出力します。これは`${output}/summary.txt`にあり、 `${output}`は`config.toml`ファイルの`output-dir`の値です。

```summary
+---------------------+--------------------+----------------+---------+-----------+
|        TABLE        | STRUCTURE EQUALITY | DATA DIFF ROWS | UPCOUNT | DOWNCOUNT |
+---------------------+--------------------+----------------+---------+-----------+
| `sbtest`.`sbtest99` | true               | +97/-97        |  999999 |    999999 |
| `sbtest`.`sbtest96` | true               | +0/-101        |  999999 |   1000100 |
+---------------------+--------------------+----------------+---------+-----------+
Time Cost: 16.75370462s
Average Speed: 113.277149MB/s
```

-   `TABLE` : 対応するデータベース名とテーブル名
-   `RESULT` : チェックが完了したかどうか。 `skip-non-existing-table = true`を設定した場合、アップストリームまたはダウンストリームに存在しないテーブルのこの列の値は`skipped`になります。
-   `STRUCTURE EQUALITY` : テーブル構造が同じかどうかをチェックします
-   `DATA DIFF ROWS` : `rowAdd` `rowDelete`テーブルを修正するために追加/削除する必要がある行の数を示します。

### 矛盾したデータを修正するための SQL ステートメント {#sql-statements-to-fix-inconsistent-data}

データチェックプロセス中に異なる行が存在する場合、それらを修正するための SQL ステートメントが生成されます。チャンク内にデータの不整合が存在する場合、 `chunk.Index`という名前の SQL ファイルが生成されます。 SQL ファイルは`${output}/fix-on-${instance}`にあり、 `${instance}`は`config.toml`ファイルの`task.target-instance`の値です。

SQL ファイルには、チャンクが属するテイルと範囲情報が含まれます。 SQL ファイルについては、次の 3 つの状況を考慮する必要があります。

-   ダウンストリーム データベース内の行が欠落している場合は、REPLACE ステートメントが適用されます。
-   ダウンストリーム データベース内の行が冗長である場合、DELETE ステートメントが適用されます。
-   ダウンストリーム データベース内の行の一部のデータに一貫性がない場合、REPLACE ステートメントが適用され、SQL ファイル内で一貫性のない列に注釈が付けられます。

```sql
-- table: sbtest.sbtest99
-- range in sequence: (3690708) < (id) <= (3720581)
/*
  DIFF COLUMNS ╏   `K`   ╏                `C`                 ╏               `PAD`
╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍
  source data  ╏ 2501808 ╏ 'hello'                            ╏ 'world'
╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍
  target data  ╏ 5003616 ╏ '0709824117-9809973320-4456050422' ╏ '1714066100-7057807621-1425865505'
╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╋╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍╍
*/
REPLACE INTO `sbtest`.`sbtest99`(`id`,`k`,`c`,`pad`) VALUES (3700000,2501808,'hello','world');
```

## ノート {#note}

-   sync-diff-inspector はデータをチェックするときに一定量のサーバーリソースを消費します。営業時間のピーク時に sync-diff-inspector を使用してデータをチェックすることは避けてください。
-   MySQL のデータと TiDB のデータを比較する前に、テーブルの照合順序構成に注意してください。主キーまたは一意キーが`varchar`タイプで、MySQL の照合構成が TiDB の照合順序構成と異なる場合、照合順序順序の問題により、最終チェック結果が正しくない可能性があります。 sync-diff-inspector 構成ファイルに照合順序を追加する必要があります。
-   sync-diff-inspector は、まず TiDB 統計に従ってデータをチャンクに分割します。統計の正確性を保証する必要があります。 TiDB サーバーの*ワークロードが軽い*場合は、 `analyze table {table_name}`コマンドを手動で実行できます。
-   `table-rules`に特に注意してください。 `schema-pattern="test1"` `table-pattern = "t_1"`設定すると`target-schema="test2"` `target-table = "t_2"` `test1`ソース データベースの`t_1`スキーマと`test2` 。対象データベース内の`t_2`スキーマが比較されます。シャーディングは sync-diff-inspector でデフォルトで有効になっているため、ソース データベースに`test2` . `t_2`表、 `test1` 。 `t_1`テーブルと`test2` 。シャーディングとして機能するソース データベース内の`t_2`テーブルは、 `test2`と比較されます。 `t_2`ターゲットデータベースのテーブル。
-   生成された SQL ファイルはデータ修復の参考としてのみ使用されるため、これらの SQL ステートメントを実行してデータを修復する前に確認する必要があります。
