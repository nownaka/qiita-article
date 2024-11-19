---
title: 【PowerShell】Sort-Object で配列の並び替え
tags:
  - "PowerShell"
  - "初心者"
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

Azure CosmosDB の Rest API を利用してデータを取得する際に `ORDER BY` など一部のクエリが利用できないようでした。
そこで データ取得後に PowerShell の処理で並べ替えを実装することにしました。

# 環境

- macOS: `15.0.1`
- PowerShell: `7.4.0`

# Sort-Object とは？

プロパティの値などを利用して、配列などのオブジェクトを並べ替えができる PowerShell のメソッドです。
（PowerShell はメソッドという表現でいいのだろうか？）

# 使い方

```powershell
Sort-Object -Property <プロパティ> -Descending
```

オプションは色々とあるみたいですが、今回は -Property と -Descending のみ指定して実行します。
これで プロパティの値に基づいて、降順に並び替えることができます。

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/sort-object?view=powershell-7.4

# 実際に使ってみる

```powershell
$sampleData = @(
  {
    id: 25,
    name: "Pikachu"
  },
  {
    id: 151,
    name: "mewtwo"
  },
  {
    id: 6,
    name: "Charizard"
  }
)

$sampleData | Sort-Object -Property id -Descending

# 実行結果：
#
# id: 151,
#   name: "mewtwo"
#
#
#   id: 25,
#   name: "Pikachu"
#
#
#   id: 6,
#   name: "Charizard"
```

ちゃんと並び替えできました。

# さいごに

今回は、PowerShell で Sort-Object の配列の並び替えを記事にしてみました。
簡単な内容でも忘れそうなので、こういったものも記事にして記憶に残そうと思います。

最後まで見ていただきありがとうございました。
誰かの役に立てば幸いです。
