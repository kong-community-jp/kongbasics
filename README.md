# OpenAPI Spec of Kong Admin API

本プロジェクトでは、Nginxベースの高速なAPI Gatewayとして有名なKongの設定を行うAdmin APIを、OpenAPI仕様のドキュメントとして整備し、そのドキュメントをブラウザ表示して、try it機能を使ってAPI実行を行い、Kong Gatewayの設定方法を学びます。

[Kong Admin API](https://david3080.github.io/kongadminapi)

下記のルールで整理しています。
- 対象オブジェクトは、Service、Route、Consumer、Plugin、Certificate、SNI、Upstreamの順
(※ [Kongにおける証明書に関するブログ](https://konghq.com/blog/mutual-tls-api-gateway)によるとKong OSSではクライアント認証を行うmTLSプラグインが提供されていないため、クライアント証明書の検証するCA Certificateオブジェクトは使わないため対象から外しています）
- HTTPメソッドは、GET、POST、PUT、PATCH、DELETEの順
- OAS1ファイル上ではpathのURIは重複できないためKongオフィシャルドキュメントのパス順ではなく、操作対象のオブジェクトごとに整理しています
- OAS内のタグはオブジェクト名もしくは2つのオブジェクト名の組み合わせで作成しています
- 上記の考え方は[Kong Admin APIのオブジェクトごとの整理](./APIList.md)にまとめています。

# Kong GW OSS V3.1 + PostgreSQLのdocker-compose.yml

上述のKong Admin APIドキュメントから「try it」を実行して学習するための環境です。

以下を参考に作成しています。

https://github.com/Kong/demo-scene/tree/main/kong-docker/postgres

Docker Composeが稼働する環境で下記のコマンドを実行するとKongが起動します。

```
$ docker-compose up -d
```

証明書が不正ですが、https(443ポート)でKong APIにアクセスでき、8001ポートでKong Admin APIにアクセスできるように設定してあります。

PostgreSQLのボリュームを削除して開始したい場合は下記のコマンドを実行します。
```
$ docker volume rm kongadminapi_kong-db-data
```

# Kong Admin API Demo
[OASドキュメント](https://david3080.github.io/kongadminapi/mockbin.html)のtry it機能を使って下記のデモシナリオを実行します。

※上述のdocker-composeを使ってKong 3.1が実行されていないとKong Admin APIを利用できないことご注意ください。

## ① mockbin　APIのサービス登録

[mockbinサイト](https://mockbin.org/request)は、HTMLやJSON、YAML、XMLといった形式でリクエストヘッダやボディをそのままエコーして返してくれるテスト用のAPIサービスです. 例えば、このURLにブラウザからアクセスするとHTML形式のコンテンツがレスポンスされますが、ヘッダに「accept: "application/json"」を設定するとJSON形式でレスポンスされます。そのため、Kongのサービスに「accept: "application/json"」ヘッダを付与するプラグインを指定して、REST APIとして機能する設定を行います.

1. [サービスの登録](https://david3080.github.io/kongadminapi/mockbin.html#/operations/1-2_create-service)
2. [ルートの登録](https://david3080.github.io/kongadminapi/mockbin.html#/operations/2-8_create-route-associated-to-a-specific-service): "service_name_or_id"に"mockbin-service"と入力して実行します。
3. [request-transformerプラグインの設定](https://david3080.github.io/kongadminapi/mockbin.html#/operations/4-8_create-plugin-associated-to-a-specific-service): "service_name_or_id"に"mockbin-service"と入力して実行します。
4. Kong経由のmockbin API実行

    下記のURLにアクセスし、レスポンスボディのJSON内のheders配列に「accept: "application/json"」が含まれていることを確認

    http://localhost:8000/mockbin

## ② mockbin　APIをターゲットに登録し、アップストリーム経由で実行する

1. [アップストリームの登録](https://david3080.github.io/kongadminapi/mockbin.html#/operations/7-2_create-upstream)

## (TODO)③ consumerとauthを登録し、Basic認証でmockbin APIを実行する
## (TODO)④ rate-limitingプラグインを設定する
## (TODO)⑤ certificateとsniを設定し、httpsでmockbin APIを実行する

# 参考資料
- [KongのオフィシャルドキュメントのAdmin APIドキュメント](https://docs.konghq.com/gateway/3.0.x/admin-api)
- [Kongのオフィシャルドキュメントの元になっているLUAファイル](https://github.com/Kong/kong/blob/master/autodoc/admin-api/data/admin-api.lua)
- [KongのオフィシャルGithubサイトに置いてある中途半端なOASファイル](https://github.com/Kong/kong/blob/master/kong-admin-api.yml)

# TODO
- [Stoplight Elementsデモサイト](https://elements-demo.stoplight.io/#/)を参考に複数のOASファイルを選択可能にする
