---
title: 【Azure】Azure CLI を利用してカスタムロールを作成・更新する
tags:
  - Azure
private: false
updated_at: "2024-11-25T07:00:32+09:00"
id: 189265b20f47174989f5
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

業務でカスタムロールを触わりました。
今後 Azure CLI を利用して管理にできたらと思いましたので、少し調べてみました。

# 環境

- macOS: `15.1.1`
- Azure CLI: `2.62.0`

# カスタムロールの作成

まずは、カスタムロールを定義した Json を準備します。
下記の例は 仮想マシンの電源管理を行うためのカスタムロールです。
割り当て可能なスコープは「サブスクリプション」にしています。
（`{subscriptionId}`の値は適宜置き換えて下さい。）

```json: vmPowerOperator@v1.json
{
  "Name": "Virtual Machine Power Operator",
  "IsCustom": true,
  "Description": "仮想マシンの電源操作ができる。(読み取り/起動/停止/再起動)",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/restart/action"
  ],
  "NotActions": [

  ],
  "assignableScopes": [
    "/subscriptions/{subscriptionId}"
  ],
}
```

Json の準備ができたら、下記コマンドで作成します。

```bash
az role definition create --role-definition vmPowerOperator@v1.json
```

#### 実行結果

```
Readonly attribute type will be ignored in class <class 'azure.mgmt.authorization.v2022_05_01_preview.models._models_py3.RoleDefinition'>
{
  "assignableScopes": [
    "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  ],
  "createdBy": null,
  "createdOn": "2024-11-24T11:55:20.045506+00:00",
  "description": "仮想マシンの電源操作ができる。(読み取り/起動/停止/再起動)",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.Authorization/roleDefinitions/e5a74e86-67d2-442c-9b2f-8c7adaad96e1",
  "name": "e5a74e86-67d2-442c-9b2f-8c7adaad96e1",
  "permissions": [
    {
      "actions": [
        "Microsoft.Compute/virtualMachines/read",
        "Microsoft.Compute/virtualMachines/start/action",
        "Microsoft.Compute/virtualMachines/deallocate/action",
        "Microsoft.Compute/virtualMachines/restart/action"
      ],
      "condition": null,
      "conditionVersion": null,
      "dataActions": [],
      "notActions": [],
      "notDataActions": []
    }
  ],
  "roleName": "Virtual Machine Power Operator",
  "roleType": "CustomRole",
  "type": "Microsoft.Authorization/roleDefinitions",
  "updatedBy": "ac96884b-7519-437a-aadf-e1e0530145ea",
  "updatedOn": "2024-11-24T11:55:20.045506+00:00"
}
```

# カスタムロールの更新

先ほど作成したカスタムロールの割り当て可能なスコープに「管理グループ」を追加してみたいと思います。
（`{managementGroupId}`は適宜置き換えて下さい。）

```json: vmPowerOperator@v2.json
{
  "Name": "Virtual Machine Power Operator",
  "IsCustom": true,
  "Description": "仮想マシンの電源操作ができる。(読み取り/起動/停止/再起動)",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/restart/action"
  ],
  "NotActions": [

  ],
  "assignableScopes": [
    "/subscriptions/{subscriptionId}",
    "/providers/Microsoft.Management/managementGroups/{managementGroupId}"
  ]
}

```

Json の更新ができたら、下記コマンドで反映させます。

```bash
az role definition update --role-definition vmPowerOperator@v2.json
```

#### 実行結果

```
Role "id" is missing. Look for the role in the current subscription...
Readonly attribute type will be ignored in class <class 'azure.mgmt.authorization.v2022_05_01_preview.models._models_py3.RoleDefinition'>
{
  "assignableScopes": [
    "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "/providers/Microsoft.Management/managementGroups/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  ],
  "createdBy": null,
  "createdOn": "2024-11-24T12:18:48.819397+00:00",
  "description": "仮想マシンの電源操作ができる。(読み取り/起動/停止/再起動)",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.Authorization/roleDefinitions/e5a74e86-67d2-442c-9b2f-8c7adaad96e1",
  "name": "e5a74e86-67d2-442c-9b2f-8c7adaad96e1",
  "permissions": [
    {
      "actions": [
        "Microsoft.Compute/virtualMachines/read",
        "Microsoft.Compute/virtualMachines/start/action",
        "Microsoft.Compute/virtualMachines/deallocate/action",
        "Microsoft.Compute/virtualMachines/restart/action"
      ],
      "condition": null,
      "conditionVersion": null,
      "dataActions": [],
      "notActions": [],
      "notDataActions": []
    }
  ],
  "roleName": "Virtual Machine Power Operator",
  "roleType": "CustomRole",
  "type": "Microsoft.Authorization/roleDefinitions",
  "updatedBy": "ac96884b-7519-437a-aadf-e1e0530145ea",
  "updatedOn": "2024-11-24T12:18:48.819397+00:00"
}
```

# さいごに

Azure CLI を利用して、カスタムロールの作成と更新をやってみました。
単発だと Portal からポチポチやるのもいいのですが、繰り返しやるとなると面倒だったりするので、できるようになってよかったです。

最後までみていただきありがとうございました。
誰かの役に立てば幸いです。

# 参考

https://learn.microsoft.com/ja-jp/azure/role-based-access-control/custom-roles

https://learn.microsoft.com/ja-jp/azure/role-based-access-control/custom-roles-cli
