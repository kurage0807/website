
# KAMONOHASHIのカスタムインストール

[インストール手順](/docs/install-and-update)では要件に合わない場合に、インストールスクリプトを使わずに構築する手順を解説します

# 手順概要

1. GPU機能を有効化したKubernetesを構築します
2. Minio NAS Gatewayを構築します
3. KAMONOHASHIを Kubernetesクラスタにインストールします

## GPU機能を有効化したKubernetesを構築
* 任意の方法でKubernetesクラスタを構築します
  * v1.14.1
  * ネットワークプラグインはweaveを使用します
  * kube-proxy mode は iptablesを使用します
* [nvidia-docker2をインストール](https://github.com/NVIDIA/nvidia-docker)します。そして、デフォルトのdockerランタイムとして設定します
* [nvidia-device-pluginをインストール](https://github.com/NVIDIA/k8s-device-plugin)します
* nfs-clientを各GPUサーバーにインストールします

## Minio NAS Gatewayを構築
* NFSサーバーを構築します
* [Minio NAS Gateway](https://docs.min.io/docs/minio-gateway-for-nas.html)を構築し、NFSサーバーのマウントポイントにMinio NAS Gatewayがデータを書き込むようにします
* KubernetesクラスタのノードにNFSクライアントをインストールし、マウントコマンドでNFSが指定できるようにします

## KAMONOHASHIを Kubernetesクラスタにインストール

### インストール準備

* Kubernetesマスターにrootユーザーでログインします
* 次のコマンドを実行し、インストールスクリプトを取得します

```
KQI_VERSION=1.1.3 
wget -O /tmp/deploy-tools-$KQI_VERSION.tar.gz https://github.com/KAMONOHASHI/kamonohashi/releases/download/$KQI_VERSION/deploy-tools-$KQI_VERSION.tar.gz
mkdir -p /var/lib/kamonohashi/deploy-tools/$KQI_VERSION/
cd /var/lib/kamonohashi/deploy-tools/$KQI_VERSION/
tar --strip=1 -xf /tmp/deploy-tools-$KQI_VERSION.tar.gz
cd kamonohashi
mkdir conf && cp -i conf-template/* conf/
```

* 上記コマンドの移動先で、`./deploy-kqi.sh prepare`を実行します
* 上記コマンドの移動先で、conf/settings.ymlを次に従って編集します

#### KAMONOHASHIインストールの設定ファイル編集

##### 必須項目
|項目|内容|例|
|:---|:---|:---|
|${INGRESS_NODE}|k8s ingress controllerの配置ノード|kqi-node1|
|${INGRESS_NODE_IP}|${INGRESS_NODE}で指定したマシンのIP|10.1.1.1|
|${VIRTUAL_HOST}|ブラウザでアクセスさせるホスト名|kamonohashi.ai|
|${KQI_NODE}|kamonohashiの各種コンテナの配置ノード|kqi-node1|
|${NODES}|GPUサーバーホスト名。,区切りで複数指定可能|gpu-node1,gpu-node2|
|${OBJECT_STORAGE}|Minioのホスト名|minio-node|
|${OBJECT_STORAGE_PORT}|Minioのポート番号|9000|
|${OBJECT_STORAGE_ACCESSKEY}|Minioのアクセスキー||
|${NFS_STORAGE}|MinioのマウントしているNFSのホスト名|nfs-node|
|${NFS_PATH}|NFSのエクスポートパス|/nfs/share|

##### オプション項目
settings.ymlのコメントアウトをはずして設定します

###### プロキシ設定
|項目|内容|例|
|:---|:---|:---|
|http_proxy|http_proxy|"http://proxy.my-corp.local"|
|https_proxy|https_proxy|"http://proxy.my-corp.local"|
|no_proxy|no_proxy。localhost,127.0.0.1,k8s,.local, Minio NAS Gatewayを必須で含める|"localhost,127.0.0.1,k8s,minio,.local,.my-corp.domain"|

###### active directory(LDAP)の設定
```
appsettings:
```
の以下のプロパティを編集します

|項目|内容|例|
|:---|:---|:---|
|ActiveDirectoryOptions__Domain|LDAPドメイン|"my-org.my-corp.co.local"|
|ActiveDirectoryOptions__BaseDn|LDAP DN|"DC=my-org,DC=my-corp,DC=co,DC=local"|
|ActiveDirectoryOptions__BaseOu|LDAP OU|"\"\"" (OUなしの場合)|
|ActiveDirectoryOptions__Server|LDAP Server|"ad01"|

###### バックアップの設定
[バックアップとリストア](/docs/how-to/infra/#バックアップとリストア)を参照してください

###### コンテナのリソースの修正
```
resources:
```
の各コンテナの以下のプロパティを編集します

|項目|内容|例|
|:---|:---|:---|
|CPU|予約するcpuリソース。k8sの指定と同じフォーマットが可能|2|
|メモリ|予約するメモリリソース。k8sの指定と同じフォーマットが可能|2Mi|

### インストール実行

* `./deploy-kqi.sh deploy`を実行します

# カスタムインストールしたKAMONOHASHIのアンインストール

* `./deploy-kqi.sh clean`を実行します
* `rm -rf /var/lib/kamonohashi`を実行し、KAMONOHASHIの保存データを削除します

# カスタムインストールしたKAMONOHASHIのバージョンアップ

インストール方法の[KAMONOHASHI Webアプリのみのバージョンアップ](/docs/install-and-update)を実施してください



