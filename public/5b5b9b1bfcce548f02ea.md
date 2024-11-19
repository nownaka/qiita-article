---
title: 【Swift】週初めの日付を取得する
tags:
  - Xcode
  - Swift
private: false
updated_at: '2022-05-19T08:55:25+09:00'
id: 5b5b9b1bfcce548f02ea
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに
ある日付からその日が含まれる週初めの日付を取得したい場面ってありませんか？
僕なりに考えて実装してみたので備忘録として残します。

## 環境
- Swift: version 5.6
- Xcode: Version 13.3.1 (13E500a)
- macOS: 12.3.1 (21E258)

## 手順

やり方はたくさんあると思いますが、今回は下記の手順で処理をします。

1. 当日の0:00の日時を取得
2. 曜日番号を取得
3. 曜日番号から日時の演算を行う
4. 週初めの日付を取得

## 実装

`extension`で`Calendar`のメソッドとして実装してみました。
```swift
import Foundation

extension Calendar {
    //月曜始まりor日曜始まり
    enum startOfWeek {
        case sunday
        case monday
    }

    func initialDayOfWeek(_ date: Date = Date(), start: startOfWeek = .sunday) -> Date {
        //当日0:00の日時を取得
        let am = self.startOfDay(for: date)
        //曜日番号を取得
        let weekNumber = self.component(.weekday, from: am)

        if start == .sunday { //日曜始まりの場合
            return self.date(byAdding: .day, value: -(weekNumber - 1), to: am)!
        } else { //月曜始まりの場合
            if weekNumber == 1 {
                return self.date(byAdding: .day, value: -6, to: am)!
            } else {
                return self.date(byAdding: .day, value: -(weekNumber - 2), to: am)!
            }
        }
    }
}
```

## 動作確認

注意点は1点。
そのまま出力すると9時間遅れになってしまうので、実際に出力する時は`DateFormatter`を使用して下さい。
```swift:
let calendar = Calendar.current

//今日
let a = calendar.initialDayOfWeek()
let b = calendar.initialDayOfWeek(start: .monday)
//5月6日
let date = calendar.date(from: DateComponents(year: 2022, month: 5, day: 6))!
let c = calendar.initialDayOfWeek(date)
let d = calendar.initialDayOfWeek(date, start: .monday)

//そのまま出力（9時間遅れ）
print(a)
print(b)
print(c)
print(d)

print("----------------------------------")

var dateFormater: DateFormatter {
    let df = DateFormatter()
    df.calendar = Calendar(identifier: .gregorian)
    df.locale = Locale(identifier: "ja_JP")
    df.timeZone = TimeZone(identifier: "Asia/Tokyo")
    df.dateStyle = .full
    df.timeStyle = .long
    return df
}
//DateFormatterで調整して出力
print(dateFormater.string(from: a))
print(dateFormater.string(from: b))
print(dateFormater.string(from: c))
print(dateFormater.string(from: d))

```

### 実行結果

```
2022-05-14 15:00:00 +0000
2022-05-15 15:00:00 +0000
2022-04-30 15:00:00 +0000
2022-05-01 15:00:00 +0000
----------------------------------
2022年5月15日 日曜日 0:00:00 JST
2022年5月16日 月曜日 0:00:00 JST
2022年5月1日 日曜日 0:00:00 JST
2022年5月2日 月曜日 0:00:00 JST
```

期待通りに出力されています。

## さいごに
週初めの日付を取得する方法をまとめてみました。
さいごまで見ていただきありがとうございました。

## 参考記事

https://capibara1969.com/2100/#toc3

https://www.choge-blog.com/programming/swiftdateandtimeweekday/

