---
title: Devcontainer で Qiita CLI を利用しようと思ったらちょっと詰まった話
tags:
  - Docker
  - devcontainer
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

少し前から [Qiita CLI](https://github.com/increments/qiita-cli) を利用して、Qiita 記事の管理を始めました。
最近 Docker 関連の勉強をしていて、 Devcontainer を知ったので Qiita 執筆環境に利用してみることに。
その際に、コンテナ内で`npx qiita preview` を実行したあと、ローカル環境からページが表示されない状況になったので備忘録として残しておこうと思います。

# 環境

- macOS: `M1 15.1.1`
- Docker: `27.3.1`
- vscode: `1.95.3`

# Devcontainer での Qiita CLI 環境構築

以下の記事で詳しくまとめられていたので、参考にさせていただきました。
コンテナ内で認証トークンを参照できなくなる問題(credential.json)の対処法についても記載があったので、助かりました。。

https://qiita.com/hyga2c/items/8618027fe771855debb4

# 発生した問題

Devcontainer の周りの設定は、スムーズに完了し、コンテナ内部で `npx qiita preview` を実行したところ、
ローカルのブラウザが立ち上がりますが、ページにアクセスできない状況になりました。

- Safari：(私の環境のデフォルトブラウザ)
  ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2513223/a26b3a6e-b071-624c-0e3f-ec3a4fe1c575.png)

## 原因

コンテナ内部で`localhost`で起動していたことが原因でき s た。
`localhost`でリッスンしていると、ローカル環境からアクセスできなくなるようです。

https://qiita.com/amuyikam/items/01a8c16e3ddbcc734a46

## 対処法

コンテナ内部で `0.0.0.0` で起動すれば良いみたいです。
Qiita CLI のホストを指定している、`qiita.config.json`を下記のように変更したらうまくいきました。

```json: qiita.config.json
{
  "includePrivate": false,
  "host": "0.0.0.0", // localhost から変更
  "port": 8888
}
```

ただこれだと、ローカルで起動したときに逆にアクセスできなくなるので別途`qiita.config.devcontainer.json`を用意して、ボリュームマウントすることで対応しました。
（もっといい方法あるかもですが。）

```docker: docker-compose.yml
# 該当部分だけ抜粋
volumes:
  - ../:/workspace:delegated
  - ../credentials.json:/root/.config/qiita-cli/credentials.json
  - ../qiita.config.devcontainer.json:/workspace/qiita.config.json
```

# さいごに

Devcontainer を利用した Qiita CLI の環境を整えることができました。
これで執筆が捗るかな？

# 参考

https://qiita.com/hyga2c/items/8618027fe771855debb4

https://qiita.com/amuyikam/items/01a8c16e3ddbcc734a46
