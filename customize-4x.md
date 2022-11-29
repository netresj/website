---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: single
classes: wide
title: "カスタマイズ設定ガイド"
permalink: /docs/install-and-update/customize-4x
sidebar:
  nav: "docs"
---

## インストールするGPUドライババージョンの指定
* `deepops/config/settings.yml`に`nvidia_driver_ubuntu_branch: <version>`のフォーマットで記載します
* 460の場合の例
```yaml
nvidia_driver_ubuntu_branch: "460"
```

## 自動構築のNFSサーバでなく既存のNFSサーバを使用する
例えばHWベンダーのNFSサーバを使用する場合にこのカスタマイズを行います。
NFSサーバはrootユーザでの読み書きが可能であることが要件です

#### 1. 次の`deepops/config/inventry`の記述を削除します

```
[nfs-server]
${NFS}
```

#### 2. `kamonohashi/conf/settings.yml`の次の箇所を編集します

```yaml
appsettings:
...
  DeployOptions__NfsStorage: "storage"
  DeployOptions__NfsPath: "/var/lib/kamonohashi/nfs"
```

|yamlの項目|内容|
|---|---|
|appsettings.DeployOptions__NfsStorage|NFSサーバのホスト名|
|appsettings.DeployOptions__NfsPath|NFSサーバのエクスポートパス|

## 自動構築のオブジェクトストレージではなく既存のものを使用する
* オブジェクトストレージとNFSは同期されている必要があります。  
NFSに書き込んだファイルはオブジェクトストレージからアクセスできる必要があります。逆も同様です。

次の設定ファイル編集を行います

#### 1. `deepops/config/inventry`のkube-nodeの項目からオブジェクトストレージに指定したマシンを削除します
* 編集前
```
[kube-node]
gpu-node1
object-storage
gpu-node2
```
* 編集後
```
[kube-node]
gpu-node1
gpu-node2
```

#### 2.  `kamonohashi/conf/settings.yml`の`minio_deploy`を`false`に指定します

```yaml
minio_deploy: false
```

#### 3. 構築完了後、kamonohashiのweb画面のストレージ設定画面から、オブジェクトストレージの設定を変更します

## insecure-registryを設定する
`deepops/config/settings.yml`に次の記述を追記します
```yaml
docker_insecure_registries: ["<host名:ポート>"]
```

## LDAP (AD: Active Directory)の設定
* LDAP 認証として使用できるのは、Active Directory(AD) のみになります。
* LDAP 認証を使用する場合、`/var/lib/kamonohashi/deploy-tools/kamonohashi/conf/settings.yml` を開きます。
* 以下のプロパティを追加します。

| プロパティ名                   | 内容          | 設定例                                |
| :----------------------------- | :------------ | :------------------------------------ |
| ActiveDirectoryOptions__Domain | LDAP ドメイン | "my-org.my-corp.co.local"             |
| ActiveDirectoryOptions__BaseDn | LDAP DN       | "DC=my-org,DC=my-corp,DC=co,DC=local" |
| ActiveDirectoryOptions__BaseOu | LDAP OU       | "\\"\\"" (OUなしの場合)               |
| ActiveDirectoryOptions__Server | LDAP サーバ   | "ad-server"                           |


* 追加例は以下になります。インデントを DeployOptions とそろえて記載してください。

```yaml
appsettings:
  DeployOptions__GpuNodes: ...
  ...
  ActiveDirectoryOptions__Domain: "my-org.my-corp.co.local"
  ActiveDirectoryOptions__BaseDn: "DC=my-org,DC=my-corp,DC=co,DC=local"
  ActiveDirectoryOptions__BaseOu: "\"\""
  ActiveDirectoryOptions__Server: "ad-server"
```

## Webhookの環境設定

* Slack 通知機能を使用する場合、`/var/lib/kamonohashi/deploy-tools/kamonohashi/conf/settings.yml` を開きます。
* 以下のプロパティを追加します。

| プロパティ名                        | 内容                                                                                                         | 設定例    |
| :---------------------------------- | :----------------------------------------------------------------------------------------------------------- | :-------- |
| ContainerManageOptions__WebEndPoint | KAMONOHASHI の Web エンドポイントを設定します。<br>設定例は KAMONOHASHI ノードのホスト名を指定する場合です。 | kqi-node1 |

* 追加例は以下になります。インデントを DeployOptions とそろえて記載してください。

```yaml
appsettings:
  DeployOptions__GpuNodes: ...
  ...
  ContainerManageOptions__WebEndPoint: "kqi-node1"
```