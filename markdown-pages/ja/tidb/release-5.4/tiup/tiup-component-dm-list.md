---
title: tiup dm list
---

# tiup dm list {#tiup-dm-list}

`tiup-dm`は、同じ制御マシンを使用した複数のクラスターのデプロイをサポートします。 `tiup dm list`コマンドを使用して、現在ログインしているユーザーがコントロールマシンを使用して展開されているクラスターを確認できます。

> **ノート：**
>
> デフォルトでは、デプロイされたクラスターのデータは`~/.tiup/storage/dm/clusters/`ディレクトリーに保管されます。現在ログインしているユーザーは、同じコントロールマシンに他のユーザーが展開しているクラスターを表示できません。

## 構文 {#syntax}

```shell
tiup dm list [flags]
```

## オプション {#options}

### -h、-help {#h-help}

-   ヘルプ情報を出力します。
-   データ型： `BOOLEAN`
-   このオプションは、デフォルトで`false`の値で無効になっています。このオプションを有効にするには、このオプションをコマンドに追加し、 `true`の値を渡すか、値を渡さないようにします。

## 出力 {#output}

次のフィールドで構成されるテーブル：

-   `Name` ：クラスタ名。
-   `User` ：クラスタをデプロイしたユーザー。
-   `Version` ：クラスタバージョン。
-   `Path` ：コントロールマシン上のクラスタ展開データのパス。
-   `PrivateKey` ：クラスタへの秘密鍵のパス。

[&lt;&lt;前のページに戻る-TiUPDMコマンドリスト](/tiup/tiup-component-dm.md#command-list)