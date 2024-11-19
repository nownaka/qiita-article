---
title: '【Swift】contains(_:)で任意の文字が含まれているか調べる'
tags:
  - Xcode
  - Swift
private: false
updated_at: '2022-05-11T11:25:22+09:00'
id: 4f53beddfb6ff6f2ee1d
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
`contains(_:)`メソッドを使う機会があったので備忘録として残します。

## 環境
- Swift: version 5.6
- Xcode: Version 13.3.1 (13E500a)
- macOS: 12.3.1 (21E258)


## `contains(_:)`メソッド

```swift
func contains(_ element: Self.Element) -> Bool
```

- Sequenceプロトコルで提供されるメソッド。
- 引数と同じ要素が含まれるかどうかを調べることができる。

## 使い方

### 例題

> 配列["MacBook", "iMac", "Mac Pro", "iPhone", "iPad", "Apple Watch"]に対して下記の操作行う。
> 1. iPhoneが含まれているか調べる。
> 2. iが含まれている製品を抽出する。 

### 実装
```swift
let product = ["MacBook", "iMac", "Mac Pro", "iPhone", "iPad", "Apple Watch"]

//iPhoneが含まれているかを調べる。
print(product.contains("iPhone"))
//true

//filterメソッドと組み合わせて、iが含まれる製品を抽出する
let filtered = product.filter { $0.contains("i") }
print(filtered)
//["iMac", "iPhone", "iPad"]
```

## `contains(where:)`メソッド

引数にクロージャを渡す`contains(where:)`もあります。

```swift
func contains(where predicate: (Self.Element) throws -> Bool) rethrows -> Bool
```
- 引数に渡したクロージャを満たす要素が含まれているかどうかを調べることができる。`

`contains(where:)`の使用例は[こちら](https://qiita.com/nownaka/items/a0d5b0d4ca30360dd6fb)。

## さいごに
`contains(_:)`メソッドの使い方をまとめてみました。
最後まで見ていただきありがとうございました。

## 参考記事・書籍

https://developer.apple.com/documentation/swift/array/2945493-contains

https://www.sbcr.jp/product/4815604073/
