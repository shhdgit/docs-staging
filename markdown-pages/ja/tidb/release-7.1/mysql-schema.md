---
title: mysql Schema
summary: Learn about the TiDB system tables.
---

# <code>mysql</code>スキーマ {#code-mysql-code-schema}

`mysql`スキーマには TiDB システム テーブルが含まれています。この設計は MySQL の`mysql`スキーマに似ており、 `mysql.user`などのテーブルを直接編集できます。また、MySQL の拡張機能である多数のテーブルも含まれています。

## 付与システムテーブル {#grant-system-tables}

これらのシステム テーブルには、ユーザー アカウントとその権限に関する付与情報が含まれています。

-   `user` : ユーザー アカウント、グローバル権限、およびその他の非権限列
-   `db` : データベースレベルの権限
-   `tables_priv` : テーブルレベルの権限
-   `columns_priv` : 列レベルの権限
-   `password_history` : パスワード変更履歴
-   `default_roles` : ユーザーのデフォルトの役割
-   `global_grants` : 動的権限
-   `global_priv` : 証明書に基づく認証情報
-   `role_edges` : 役割間の関係

## サーバー側のヘルプ システム テーブル {#server-side-help-system-tables}

現在、 `help_topic`は NULL です。

## 統計システムテーブル {#statistics-system-tables}

-   `stats_buckets` : 統計のバケット
-   `stats_histograms` : 統計のヒストグラム
-   `stats_top_n` : 統計のトップN
-   `stats_meta` : 総行数や更新された行などのテーブルのメタ情報
-   `stats_extended` : 拡張統計 (列間の順序相関など)
-   `stats_feedback` : 統計のクエリフィードバック
-   `stats_fm_sketch` : 統計列のヒストグラムの FMSketch 分布
-   `analyze_options` : 各テーブルのデフォルトの`analyze`オプション
-   `column_stats_usage` : 列統計の使用法
-   `schema_index_usage` : インデックスの使用
-   `analyze_jobs` : 進行中の統計収集タスクと過去 7 日間の履歴タスク レコード

## 実行計画関連のシステムテーブル {#execution-plan-related-system-tables}

-   `bind_info` : 実行計画のバインディング情報
-   `capture_plan_baselines_blacklist` : 実行計画の自動バインドのブロックリスト

## GC ワーカー システム テーブル {#gc-worker-system-tables}

-   `gc_delete_range` : 削除するKV範囲
-   `gc_delete_range_done` : 削除された KV 範囲

## キャッシュされたテーブルに関連するシステム テーブル {#system-tables-related-to-cached-tables}

-   `table_cache_meta`キャッシュされたテーブルのメタデータを保存します。

## TTL関連のシステムテーブル {#ttl-related-system-tables}

-   `mysql.tidb_ttl_table_status`すべての TTL テーブルに対して以前に実行された TTL ジョブと進行中の TTL ジョブ
-   `mysql.tidb_ttl_task`現在進行中の TTL サブタスク
-   `mysql.tidb_ttl_job_history`過去 90 日間の TTL タスクの実行履歴

## その他のシステムテーブル {#miscellaneous-system-tables}

-   `GLOBAL_VARIABLES` : グローバル システム変数テーブル

<CustomContent platform="tidb">

-   `tidb` : TiDB 実行時のバージョン情報を記録します`bootstrap`
-   `expr_pushdown_blacklist` : 式プッシュダウンのブロックリスト
-   `opt_rule_blacklist` : 論理最適化ルールのブロックリスト
-   `table_cache_meta` : キャッシュされたテーブルのメタデータ

</CustomContent>
