---
title: tiup dm import
---

# tiup dm import <span class="version-mark">DM v1.0 のアップグレード専用</span> {#tiup-dm-import-span-class-version-mark-only-for-upgrading-dm-v1-0-span}

<Note>このコマンドは、DM クラスターを v1.0 から v2.0 以降のバージョンにアップグレードする場合にのみ使用されます。</Note>

DM v1.0 では、クラスターは基本的に TiDB Ansible を使用してデプロイされます。 TiUP DM には、v1.0 クラスターをインポートし、クラスターを DM v2.0 に再デプロイするための`import`コマンドが用意されています。

> **ノート：**
>
> -   このコマンドは、DM v1.0 クラスターからの DM Portal コンポーネントのインポートをサポートしていません。
> -   クラスターをインポートする前に、まず元のクラスターの実行を停止します。
> -   v2.0 にアップグレードする必要があるデータ移行タスクの場合は、これらのタスクで`stop-task`を実行しないでください。
> -   このコマンドは、DM v2.0.0-rc.2 以降のバージョンへのインポートのみをサポートしています。
> -   `import`コマンドは、DM v1.0 クラスターを新しい DM v2.0 クラスターにインポートするために使用されます。データ移行タスクを既存の v2.0 クラスターにインポートする必要がある場合は、 [TiDB データ移行を v1.0.x から v2.0+ に手動でアップグレードする](/dm/manually-upgrade-dm-1.0-to-2.0.md)を参照してください。
> -   一部のコンポーネントのデプロイメント ディレクトリは、元のクラスターのものと異なる場合があります。 `display`コマンドで確認できます。
> -   クラスターをインポートする前に、 `tiup update --self && tiup update dm`を実行してTiUP DMコンポーネントを最新バージョンにアップグレードします。
> -   クラスターがインポートされた後は、クラスター内に DM マスター ノードが 1 つだけ存在します。 [`scale out`コマンド](/tiup/tiup-component-dm-scale-out.md)を参照して、DM マスター ノードをスケールアウトできます。

## 構文 {#syntax}

```shell
tiup dm import [flags]
```

## オプション {#options}

### -v、--クラスターバージョン {#v-cluster-version}

-   再デプロイするバージョン番号を指定します。 v2.0.0-rc.2 (v2.0.0-rc.2 を含む) 以降のバージョンを使用する必要があります。
-   データ型: `STRING`
-   このオプションはコマンドを実行するために**必要**です。

### -d、--dir {#d-dir}

-   TiDB Ansible のディレクトリを指定します。
-   データ型: `STRING`
-   このオプションがコマンドで指定されていない場合、現在のディレクトリがデフォルトのディレクトリになります。

### &#x20;--inventory {#inventory}

-   Ansible インベントリー ファイルの名前を指定します。
-   データ型: `STRING`
-   このオプションがコマンドで指定されていない場合、デフォルトのファイル名は`"inventory.ini"`です。

### --rename {#rename}

-   インポートされたクラスターの名前を変更します。
-   データ型: `STRING`
-   このオプションがコマンドで指定されていない場合、デフォルトのクラスター名はインベントリ ファイルで指定された`cluster_name`になります。

### -h, --help {#h-help}

-   ヘルプ情報を出力します。
-   データ型: `BOOLEAN`
-   このオプションは、値`false`を指定するとデフォルトで無効になります。このオプションを有効にするには、このオプションをコマンドに追加し、値`true`渡すか、値を渡しません。

## 出力 {#outputs}

インポートプロセスのログ。

[&lt;&lt; 前のページに戻る - TiUP DMコマンド一覧](/tiup/tiup-component-dm.md#command-list)
