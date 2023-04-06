---
title: TiDB Lightning Prechecks
summary: This document describes the checks that TiDB Lightning performs before performing a data migration task. These precheckes ensure that TiDB Lightning can perform the task smoothly.
---

# TiDB Lightning事前チェック {#tidb-lightning-prechecks}

TiDB 5.3.0 以降、 TiDB Lightningは、移行タスクを実行する前に構成をチェックする機能を提供します。デフォルトで有効になっています。この機能は、ディスク容量と実行構成の定期的なチェックを自動的に実行します。主な目的は、後続のインポート プロセス全体がスムーズに進むようにすることです。

次の表に、各チェック項目と詳細な説明を示します。

| チェック項目                    | 対応バージョン | 説明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| クラスタのバージョンとステータス          | = 5.3.0 | クラスタが接続できる構成か、TiKV/PD/ TiFlashのバージョンが物理インポートモードに対応しているかを確認してください。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| 権限                        | = 5.3.0 | データ ソースがクラウド ストレージ (Amazon S3) の場合、 TiDB Lightningに必要な権限があるかどうかを確認し、権限がないためにインポートが失敗しないことを確認します。                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ディスクスペース                  | = 5.3.0 | ローカル ディスクと TiKV クラスターに、データをインポートするための十分なスペースがあるかどうかを確認します。 TiDB Lightningはデータ ソースをサンプリングし、サンプル結果からインデックス サイズのパーセンテージを推定します。推定にはインデックスが含まれているため、ソース データのサイズがローカル ディスクの使用可能な領域よりも小さい場合がありますが、それでもチェックは失敗します。物理的なインポート モードでは、外部の並べ替えをローカルで行う必要があるため、 TiDB Lightningはローカル ストレージが十分かどうかもチェックします。 TiKV クラスター スペースとローカル ストレージ スペース ( `sort-kv-dir`で制御) の詳細については、 [ダウンストリームのストレージ容量要件](/tidb-lightning/tidb-lightning-requirements.md#storage-space-of-the-target-database)と[リソース要件](/tidb-lightning/tidb-lightning-physical-import-mode.md#environment-requirements)を参照してください。 |
| リージョン分布状況                 | = 5.3.0 | TiKV クラスター内のリージョンが均等に分散されているかどうか、および空のリージョンが多すぎるかどうかを確認します。空のリージョンの数が max(1000, テーブル数 * 3) を超える場合、つまり、「1000」または「テーブル数の 3 倍」のいずれか大きい方より大きい場合、インポートは実行できません。                                                                                                                                                                                                                                                                                                                                                                                                              |
| データ ファイル内の非常に大きな CSV ファイル | = 5.3.0 | バックアップ ファイルに 10 GiB を超える CSV ファイルがあり、自動スライスが有効になっていない (StrictFormat=false) 場合、インポートのパフォーマンスに影響します。このチェックの目的は、データが正しい形式であることを確認し、自動スライスを有効にすることを確認することです。                                                                                                                                                                                                                                                                                                                                                                                                                |
| ブレークポイントからの回復             | = 5.3.0 | このチェックにより、ブレークポイント リカバリ プロセス中に、データベース内のソース ファイルまたはスキーマに変更が加えられないことが保証されます。これにより、間違ったデータがインポートされます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| 既存のテーブルにインポートする           | = 5.3.0 | 作成済みのテーブルにインポートする場合、ソース ファイルが既存のテーブルと一致するかどうかを可能な限りチェックします。列数が一致しているかどうかを確認します。ソース ファイルに列名がある場合は、列名が一致するかどうかを確認します。ソース ファイルに既定の列がある場合、既定の列に既定値があるかどうかがチェックされ、既定値がある場合はチェックに合格します。                                                                                                                                                                                                                                                                                                                                                                                       |
| ターゲット テーブルが空かどうか          | = 5.3.1 | ターゲット テーブルが空でない場合、 TiDB Lightningはエラーで自動的に終了します。並行輸入モードが有効な場合 ( `incremental-import = true` )、このチェック項目はスキップされます。                                                                                                                                                                                                                                                                                                                                                                                                                                                        |