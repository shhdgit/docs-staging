---
title: Overview for Analyzing and Tuning Performance
summary: Learn about how to analyze and tune SQL performance in TiDB Cloud.
---

# パフォーマンスの分析とチューニングの概要 {#overview-for-analyzing-and-tuning-performance}

このドキュメントでは、 TiDB Cloudでの SQL パフォーマンスの分析と調整に役立つ手順について説明します。

## ユーザーの応答時間 {#user-response-time}

ユーザー応答時間は、アプリケーションがリクエストの結果をユーザーに返すまでにかかる時間を示します。次のシーケンス タイミング図からわかるように、一般的なユーザー リクエストの時間には次の時間が含まれます。

-   ユーザーとアプリケーション間のネットワークレイテンシー
-   アプリケーションの処理時間
-   アプリケーションとデータベース間の対話中のネットワークレイテンシー
-   データベースのサービス時間

ユーザーの応答時間は、ネットワークのレイテンシーと帯域幅、同時ユーザーの数とリクエストの種類、サーバーのCPU と I/O のリソース使用量など、リクエスト チェーン上のさまざまなサブシステムの影響を受けます。システム全体を効果的に最適化するには、まずユーザーの応答時間のボトルネックを特定する必要があります。

指定した時間範囲 ( `ΔT` ) 内のユーザー応答時間の合計を取得するには、次の式を使用できます。

`ΔT`の合計ユーザー応答時間 = 平均 TPS (1 秒あたりのトランザクション数) x 平均ユーザー応答時間 x `ΔT` 。

![user\_response\_time](https://download.pingcap.com/images/docs/performance/user_response_time_en.png)

## ユーザーの応答時間とシステムのスループットの関係 {#relationship-between-user-response-time-and-system-throughput}

ユーザー応答時間は、サービス時間、キュー時間、およびユーザー要求を完了するまでの同時待機時間で構成されます。

```
User Response time = Service time + Queuing delay + Coherency delay
```

-   サービス時間: リクエストの処理時にシステムが特定のリソースで消費する時間。たとえば、SQL リクエストを完了するためにデータベースが消費する CPU 時間。
-   キュー遅延: リクエストを処理するときに、システムが特定のリソースのサービスをキューで待機する時間。
-   コヒーレンシ遅延: システムがリクエストの処理時に共有リソースにアクセスできるように、他の同時タスクと通信および共同作業する時間。

システム スループットは、システムが 1 秒あたりに完了できるリクエストの数を示します。通常、ユーザーの応答時間とスループットは互いに反比例します。スループットが増加すると、システム リソースの使用率と、要求されたサービスのキューレイテンシーそれに応じて増加します。リソース使用率が特定の変曲点を超えると、キューのレイテンシーが大幅に増加します。

たとえば、OLTP ロードを実行しているデータベース システムの場合、CPU 使用率が 65% を超えると、CPU キューのスケジューリングレイテンシーが大幅に増加します。これは、システムの同時リクエストが完全に独立しているわけではなく、これらのリクエストが共有リソースをめぐって連携したり競合したりする可能性があるためです。たとえば、異なるユーザーからのリクエストが、同じデータに対して相互に排他的なロック操作を実行する場合があります。リソース使用率が増加すると、キューイングおよびスケジューリングのレイテンシーも増加します。これにより、共有リソースが時間内に解放されなくなり、他のタスクによる共有リソースの待ち時間が長くなります。

## ユーザー応答時間のボトルネックをトラブルシューティングする {#troubleshoot-bottlenecks-in-user-response-time}

TiDB Cloudコンソールには、ユーザーの応答時間のトラブルシューティングに役立つページがいくつかあります。

-   **概要**: このタブでは、合計 QPS、レイテンシー、接続、リクエスト QPS、リクエスト期間、storageサイズ、CPU、IO 読み取り、および IO 書き込みなどの TiDB メトリクスを表示できます。
-   **SQL診断**:

    -   **SQL ステートメントを**使用すると、ページ上の SQL 実行を直接観察でき、システム テーブルをクエリせずにパフォーマンスの問題を簡単に特定できます。 SQL ステートメントをクリックすると、トラブルシューティングと分析のためにクエリの実行プランをさらに表示できます。 SQL パフォーマンス チューニングの詳細については、 [SQLチューニングの概要](/tidb-cloud/tidb-cloud-sql-tuning-overview.md)を参照してください。
    -   **Key Visualizer は、** TiDB のデータ アクセス パターンとデータ ホットスポットを観察するのに役立ちます。

追加のメトリクスが必要な場合は、 [PingCAP サポート チーム](/tidb-cloud/tidb-cloud-support.md)にお問い合わせください。

レイテンシーとパフォーマンスの問題が発生した場合は、次のセクションの分析とトラブルシューティングの手順を参照してください。

### TiDB クラスター外部のボトルネック {#bottlenecks-outside-the-tidb-cluster}

**[概要]**タブでレイテンシ (P80) を観察します。この値がユーザー応答時間の P80 値よりも大幅に低い場合は、主なボトルネックが TiDB クラスターの外側にある可能性があると判断できます。この場合、次の手順を使用してボトルネックをトラブルシューティングできます。

1.  [「概要」タブ](/tidb-cloud/monitor-tidb-cluster.md)の左側で TiDB バージョンを確認します。 v6.0.0 以前のバージョンの場合は、 [PingCAP サポート チーム](/tidb-cloud/tidb-cloud-support.md)に問い合わせて、準備済みプラン キャッシュ、Raft エンジン、および TiKV AsyncIO 機能を有効にできるかどうかを確認することをお勧めします。これらの機能を有効にし、アプリケーション側の調整を行うと、スループット パフォーマンスが大幅に向上し、レイテンシーとリソース使用率が削減されます。
2.  必要に応じて、TiDB トークンの制限を増やしてスループットを向上させることができます。
3.  準備されたプランのキャッシュ機能が有効であり、ユーザー側で JDBC を使用する場合は、次の構成を使用することをお勧めします。

    ```
    useServerPrepStmts=true&cachePrepStmts=true& prepStmtCacheSize=1000&prepStmtCacheSqlLimit=20480&useConfigs=maxPerformance
    ```

    JDBC を使用せず、現在の TiDB クラスターのプリペアド プラン キャッシュ機能を最大限に活用したい場合は、クライアント側でプリペアドステートメントオブジェクトをキャッシュする必要があります。 StmtPrepare および StmtClose の呼び出しをリセットする必要はありません。各クエリで呼び出されるコマンドの数を 3 から 1 に減らします。パフォーマンス要件とクライアント側の変更の量に応じて、ある程度の開発作業が必要になります。 [PingCAP サポート チーム](/tidb-cloud/tidb-cloud-support.md)に相談してください。

### TiDB クラスターのボトルネック {#bottlenecks-in-the-tidb-cluster}

パフォーマンスのボトルネックが TiDB クラスター内にあると判断した場合は、次のことを実行することをお勧めします。

-   遅い SQL クエリを最適化します。
-   ホットスポットの問題を解決します。
-   クラスターをスケールアウトして容量を拡張します。

#### 遅い SQL クエリを最適化する {#optimize-slow-sql-queries}

SQL パフォーマンス チューニングの詳細については、 [SQLチューニングの概要](/tidb-cloud/tidb-cloud-sql-tuning-overview.md)を参照してください。

#### ホットスポットの問題を解決する {#resolve-hotstpot-issues}

ホットスポットの問題は[「キー ビジュアライザー」タブ](/tidb-cloud/tune-performance.md#key-visualizer)で確認できます。次のスクリーンショットは、ヒート マップのサンプルを示しています。地図の横座標は時刻、縦座標は表と索引です。明るい色はトラフィックが多いことを示します。ツールバーで読み取りトラフィックまたは書き込みトラフィックの表示を切り替えることができます。

![Hotspot issues](https://download.pingcap.com/images/docs/tidb-cloud/tidb-cloud-troubleshoot-hotspot.png)

次のスクリーンショットは、書き込みホットスポットの例を示しています。書き込みフロー グラフに明るい斜線 (斜め上または斜め下) が表示され、書き込みトラフィックは線の端にのみ表示されます。テーブルリージョンの数が増加するにつれて、階段状のパターンになります。これは、テーブル内に書き込みホットスポットがあることを示します。書き込みホットスポットが発生した場合は、自己インクリメント主キーを使用しているか、主キーを使用していないか、または時間依存の挿入ステートメントまたはインデックスを使用しているかを確認する必要があります。

![Write hotspot](https://download.pingcap.com/images/docs/tidb-cloud/tidb-cloud-troubleshoot-write-hotspot.png)

次のスクリーンショットに示すように、読み取りホットスポットは通常、ヒート マップ内で明るい水平線として表され、通常は多数のクエリを含む小さなテーブルになります。

![Read hotspot](https://download.pingcap.com/images/docs/tidb-cloud/tidb-cloud-troubleshoot-read-hotspot-new.png)

次のスクリーンショットに示すように、強調表示されたブロックの上にマウスを移動すると、トラフィックが多いテーブルまたはインデックスが表示されます。

![Hotspot index](https://download.pingcap.com/images/docs/tidb-cloud/tidb-cloud-troubleshoot-hotspot-index.png)

#### 規格外 {#scale-out}

クラスター[概要](/tidb-cloud/monitor-tidb-cluster.md)ページで、storage容量、CPU 使用率、TiKV IO レートのメトリックを確認します。いずれかのクラスターが長期間にわたって上限に達している場合は、現在のクラスター サイズではビジネス要件を満たせない可能性があります。クラスターをスケールアウトする必要があるかどうかを確認するには、 [PingCAP サポート チーム](/tidb-cloud/tidb-cloud-support.md)に問い合わせることをお勧めします。

#### その他の問題 {#other-issues}

上記の方法でパフォーマンスの問題を解決できない場合は、 [PingCAP サポート チーム](/tidb-cloud/tidb-cloud-support.md)に問い合わせてサポートを求めることができます。トラブルシューティング プロセスを迅速化するために、次の情報を提供することをお勧めします。

-   クラスターID
-   発行間隔と同等の通常間隔
-   問題となる現象と予想される動作
-   読み取りまたは書き込みの比率や主な動作などのビジネス ワークロードの特性

## まとめ {#summary}

一般に、次の最適化方法を使用して、パフォーマンスの問題を分析および解決できます。

| アクション                              | 効果                                                                                                  |
| :--------------------------------- | :-------------------------------------------------------------------------------------------------- |
| 準備されたプラン キャッシュ + JDBC              | スループット パフォーマンスが大幅に向上し、レイテンシーが大幅に短縮され、TiDB の平均 CPU 使用率が大幅に削減されます。                                    |
| TiKV で AsyncIO と Raft エンジンを有効にする   | スループットパフォーマンスがいくらか向上します。有効にするには、 [PingCAP サポート チーム](/tidb-cloud/tidb-cloud-support.md)に連絡する必要があります。 |
| クラスター化インデックス                       | スループットパフォーマンスが大幅に向上します。                                                                             |
| TiDB ノードをスケールアウトする                 | スループットパフォーマンスが大幅に向上します。                                                                             |
| クライアント側の最適化。 1 つの JVM を 3 つに分割     | スループット パフォーマンスは大幅に向上し、さらに分割するとスループット キャパシティがさらに向上し続ける可能性があります。                                      |
| アプリケーションとデータベース間のネットワークレイテンシーを制限する | ネットワークレイテンシーが長いと、スループットが低下し、レイテンシーが増加する可能性があります。                                                    |

将来的には、 TiDB Cloud は、より多くの観察可能なメトリクスと自己診断サービスを導入する予定です。パフォーマンス指標についてのより包括的な理解と、エクスペリエンスを向上させるための運用上のアドバイスが提供されます。
