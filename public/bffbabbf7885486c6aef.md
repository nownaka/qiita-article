---
title: 【Azure】CloudShell の使い方
tags:
  - Azure
  - CloudShell
private: false
updated_at: '2024-10-30T22:34:07+09:00'
id: bffbabbf7885486c6aef
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

CloudShell とは、文字通りクラウド上で利用できる Shell 環境です。
Azure 関連のモジュールはデフォルトでインストールされているため、手早くコマンドを試したいときには非常に便利です。

今回は Azure 環境を始めて触る人向けに CloudShell の起動方法をまとめたいと思います。

あまり困る部分はないと思いますが、はじめて Azure を触る人はちょっと不安な部分があるかもなので、参考にしてください。

# CloudShell の起動方法

### STEP 1 : [Azure Portal](https://portal.azure.com) にログインする
    
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2513223/4ef92f0f-e9c6-40b0-6cb4-207f99daa6ed.png)

### STEP 2 : 画面上部の CloudShell のアイコンを選択する
    
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2513223/e8a80e00-99c3-6f7a-c216-757c19438b5b.png)

    
### STEP 3 : シェルの種類を選択する
    
    お好きな方を選択してください。今回は Bash を選択します。
    
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2513223/7e3ebb21-c018-1a82-20c5-de497c4464da.png)

    
### STEP 4 : CloudShell の設定をする
    
    パラメータがいくつかありますが、下記の通りに選択してください。
    少し前までは、ストレージカウントとの紐付けが求められていましたが、無くなったので、より気軽に実行できるようになりました。
    
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2513223/90a3d545-2f00-146e-210b-733e8e74363c.png)

これで起動は完了です。少し待つと、コマンドが実行できるようになります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2513223/61dc0d1f-c353-b469-4eba-18b823ed5154.png)


# さいごに

CloudShell の起動方法をまとめました。（2024/10/28時点）

課金とかは気をつけながら、いろんなコマンドを実行してみましょう！
