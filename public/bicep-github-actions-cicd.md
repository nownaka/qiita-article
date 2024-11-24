---
title: 【Azure × GitHub】インフラの CI / CD を試してみる
tags:
  - Azure
  - CICD
  - GitHubActions
  - Bicep
private: false
updated_at: '2024-11-25T07:00:32+09:00'
id: 35a3782eda7ab36aa03e
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

最近 GitHub Action を利用して アプリの CI / CD 環境を構成する方法を勉強しました。
Azure Bicep と GitHub ワークフローを利用すれば、インフラも CI/CD ができる、とのことで試してみました。

# 環境

- macOS: `15.1.1`
- Visual Studio Code: `1.95.3`
- Azure CLI: `2.62.0`

# To-Do

ざっくりとやることをまとめると下記の通りになります。

1. 新規リポジトリの作成
2. リソースグループの作成
3. アプリの登録とサービスプリンシパルの作成
4. ロールの割り当て
5. フェデレーション資格情報の設定
6. GitHub にシークレットを登録
7. デプロイテンプレートの作成
8. GitHub Action 用のワークフローを作成

# 実装

今回はお試しで Vnet を 1 つ CI/CD で作成・管理できるようにしていきたいと思います。

## 1. 新規リポジトリの作成

ここは説明不要かと思います。
下記の手順で新しいリポジトリを作成します。

https://docs.github.com/ja/repositories/creating-and-managing-repositories/creating-a-new-repository

今回は、「azure-cicd-sample」というリポジトリを作成しました。
![image-0.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2513223/029880dc-65ee-2bc1-3444-9cc30debab3c.png)

## 2. リソースグループの作成

リソースグループの準備がまだの場合は作成します。
既存のリソースグループがある場合はスキップしても問題ありません。

```bash
# リソースグループ名
resourceGroupName=rg-githubcicd-sample-001
# リソースグループのリージョン
location=japaneast


# Azure ログイン
az login

# リソースグループの作成
resourceGroupId=$(az group create --location $location --name $resourceGroupName --query id --output tsv)
```

## 3. アプリの登録とサービスプリンシパルの作成

作業ユーザーには下記の権限が必要です。

- Entra ロール: **アプリケーション管理者**

```bash
# アプリの名前
appName=github_action_for_azure-cicd-sample


# アプリの登録
appId=$(az ad app create --display-name $appName --query appId --output tsv)

# サービスプリンシパルの作成
spObjectId=$(az ad sp create --id $appId --query id --output tsv)
```

## 4. ロールの割り当て

作業ユーザーには下記の権限が必要です。

- Azure ロール: 対象スコープでの**所有者**もしくは**ユーザーアクセス管理者**

また、組み込みのロール名と ID については下記で確認できます。

https://learn.microsoft.com/ja-jp/azure/role-based-access-control/built-in-roles

```bash
# ロール：ネットワーク共同作成者
roleNameOrId=4d97b98b-1d4f-4787-a291-c67834d212e7
# スコープ：先ほど作成したリソースグループ
scope=$resourceGroupId
# 割り当て先：先ほど作成したサービスプリンシパル
assignee=$spObjectId


# ロール割り当て
az role assignment create --role $roleNameOrId --assignee $assignee --scope $scope
```

## 5. フェデレーション資格情報の設定

Entra ID に作成したアプリが GitHub Action を動かせるようにフェデレーション資格情報の設定を行います。

```bash
# GitHubアカウント名
githubAccountName='{your github account name}'
# リポジトリ名
githubRepositoryName=azure-cicd-sample
# フェデレーション資格情報を設定する対象
id=$appId


# フェデレーション資格情報の定義
federation='{
    "name": "github_federation_for_{githubRepositoryName}",
    "issuer": "https://token.actions.githubusercontent.com",
    "subject": "repo:{githubAccountName}/{githubRepositoryName}:ref:refs/heads/main",
    "audiences": ["api://AzureADTokenExchange"]
}'
federation=${federation//'{githubAccountName}'/$githubAccountName}
federation=${federation//'{githubRepositoryName}'/$githubRepositoryName}

# フェデレーション資格情報の設定
az ad app federated-credential create --id $id --parameters $federation
```

## 6. GitHub にシークレットを登録

下記を GitHub リポジトリのシークレットとして登録します。

| No  | シークレット              | 説明                                     |
| --- | ------------------------- | ---------------------------------------- |
| 1   | AZURE_TENANT_ID           | Entra ID のテナント GUID                 |
| 2   | AZURE_SUBSCRIPTION_ID     | Azure サブスクリプション ID              |
| 3   | AZURE_CLIENT_ID           | クライアント ID。アプリ登録の際に生成。  |
| 4   | AZURE_RESOURCE_GROUP_NAME | リソースをデプロイするリソースグループ名 |

値の取得は下記コマンドで可能です。

```bash
# AZURE_TENANT_ID
az account show --query tenantId -o tsv
# AZURE_SUBSCRIPTION_ID
az account show --query id -o tsv
# AZURE_CLIENT_ID
echo $appId
# AZURE_RESOURCE_GROUP_NAME(この手順で新しく作成した場合)
echo $resourceGroupName
```

![image-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2513223/7901b396-611a-9e2d-bc79-149dffbc2c66.png)

## 7. デプロイテンプレートの作成

Vnet を 1 つ作成するテンプレートを作成します。

### Bicep テンプレート

```bicep: main.bicep
// --------------------------------------------------------------------------------
// Params
// --------------------------------------------------------------------------------
param virtualNetworkName string
param location string = resourceGroup().location
param addressPrefixes string[]

// --------------------------------------------------------------------------------
// Resources
// --------------------------------------------------------------------------------
resource virtualNetworks 'Microsoft.Network/virtualNetworks@2023-09-01' = {
  name: virtualNetworkName
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: addressPrefixes
    }
  }
}

```

```bicep: main.bicepparam
using './main.bicep'

param virtualNetworkName = 'githubcicd-idea-001'
param addressPrefixes = ['10.0.0.0/24']
```

## 8. GitHub Action 用のワークフローを作成

main ブランチに push された場合に、デプロイのワークフローが動くように作成します。

```yml: .github/workflows/bicep_auto_deployment.yml
name: Azure Bicep Deployment.

# トリガー
on:
  push:
    branches:
      - main

  # 手動
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      # タイムスタンプを生成
      - name: Set timestamp
        env:
          TZ: "Asia/Tokyo"
        run: echo "TIMESTAMP=$(date +'%Y%m%d_%H%M%S')" >> $GITHUB_ENV

      # チェックアウト
      - name: Checkout to the branch
        uses: actions/checkout@v4

      # Azure ログイン
      - name: Azure Login
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      # リソースデプロイ
      - name: Deploy with Azure Bicep
        uses: azure/cli@v2
        with:
          azcliversion: latest
          inlineScript: |
            az deployment group create \
              --name Deploy_${{ env.TIMESTAMP }} \
              --resource-group ${{ secrets.AZURE_RESOURCE_GROUP_NAME }} \
              --template-file main.bicep \
              --parameters main.bicepparam
```

# 動作確認

作成したファイルをコミット、プッシュして期待通りに動くか確認します。
うまくいけば下記のようになります。

#### GitHub Action

![image-2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2513223/a8990bc0-0271-0aa0-d386-26a7c521cbbf.png)

#### Azure リソースグループ デプロイ結果

![image-3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2513223/b432a59e-179e-2a15-f4c3-733cdafb40b3.png)

# さいごに

今回は、Azure Bicep と GitHub Action を利用して インフラの CI/CD を試してみました。
Azure CLI を使った操作にも慣れていきたいところです。

最後まで見ていただきありがとうございました。
誰かの役に立てば幸いです。

### テンプレート

本記事で実施した手順は、リポジトリとして保存・公開しています。
よければ参考にしてください。

https://github.com/nownaka/azure-cicd-sample

# 参考

https://learn.microsoft.com/ja-jp/azure/azure-resource-manager/bicep/deploy-github-actions?tabs=CLI%2Cuserlevel

https://learn.microsoft.com/ja-jp/entra/identity-platform/app-objects-and-service-principals?tabs=browser

https://learn.microsoft.com/ja-jp/cli/azure/ad/app?view=azure-cli-latest

https://learn.microsoft.com/en-us/cli/azure/format-output-azure-cli?tabs=bash

https://learn.microsoft.com/ja-jp/cli/azure/role/assignment?view=azure-cli-latest#az-role-assignment-create

https://learn.microsoft.com/ja-jp/azure/role-based-access-control/built-in-roles

https://jpazureid.github.io/blog/azure-active-directory/enterprise-applications-app-registrations/
