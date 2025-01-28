---
title: "【Flutter】Clineで日記アプリを作ってみた"
emoji: "📖"
type: "tech"
topics: [flutter,dart,cline,llm]
published: true
---

昨日から[Cline](https://github.com/cline/cline)を使い始めたので勉強がてらにFlutterアプリを一から作らせてみました。
Cline 3.1.0、DeepSeek Chat を使用しました。

:::message
本記事には続編があります。

【Flutter】 Clineと DeekSeek R1 で日記アプリを作ってみた
https://zenn.dev/hyshu/articles/57aedca93235b5
:::

# 準備
## Flutterプロジェクトの作成
```
flutter create calendar_example --platforms=ios,android
```

## 無駄なファイルを削除
なるべくAIにノイズが混じらないように以下のファイルを削除します
* `analysis_options.yaml`
* `calendar_example.iml`
* `pubspec.lock`
* `README.md`
* `test/widget_test.dart`

`ios`と`android`ディレクトリーも別の場所に退避しておきます。
これでファイルとディレクトリーは以下の三つだけになります。
1. `lib/main.dart`
2. `pubspec.yaml`
3. `test`

## `main.dart`と`pubspec.yaml`を編集
```dart:main.dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) => MaterialApp(home: Scaffold());
}
```

`pubspec.yaml`には必要なパッケージをあらかじめ追加しておきます。
今回はカレンダーが中心の日記アプリなので`table_calendar`を使用。
保存は安定性があるのと、SQL文もちゃんと書いてくれるか気になったので`sqflite`を使用することにしました。
```dart:pubspec.yaml
name: calendar_example
description: ""
publish_to: "none"
version: 1.0.0+1

environment:
  sdk: ^3.6.0

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  sqflite: ^2.4.1
  table_calendar: ^3.2.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0

flutter:
  uses-material-design: true
```

## lib以下にディレクトリー作成
今回の希望の設計になるようにディレクトリーを作成します。
```
lib
├── application
│   └── use_case
├── domain
│   └── entity
├── infrastructure
│   └── db
├── main.dart
└── presentation
    ├── component
    ├── inherited_widget
    └── page
```

# プロンプト
```
Flutterで日記アプリを作成してください。

要件
・pubspec.yamlの編集は禁止です。既に追加しているパッケージのみを使用してください。
・ディレクトリーの新規作成は禁止です。libのディレクトリー構成に合わせたアーキテクチャーで作成してください。
・UseCase、Entity、StoreをViewに受け渡すにはInheritedWidgetを使用してください。
・書いた日記はカレンダーで日付ごとに読めるようにします。カレンダーは月単位、週単位で切り替えられるようにしてください。週単位では日記の内容が一覧で見れます。
・カレンダーの日付をタップするとカレンダーの下に日記を表示します。
・日記は編集と削除が出来ます。削除する時は確認ダイアログを表示してください。
・本日または本日より前であれば日記が作成できます。
・日記はsqfliteでdiaryテーブルに保存します。
・WidgetテストとUnitテストを行ってください。
```

* 余計なパッケージが追加されたりしないように`pubspec.yaml`は編集させない
* アプリの設計はディレクトリーで指示する
* Flutter標準のDI手法である`InheritedWidget`を使用させる。書くのに手間がかかって中々使われない`InheritedWidget`ですが、AIなら気にしなくて良いので

上記のプロンプトをGPT-4oで英訳したものを使用します。
:::details 英語版
```
Create a diary app in Flutter.

Requirements:
- Editing pubspec.yaml is prohibited. Use only the packages already added.
- Creating new directories is prohibited. Follow the directory structure in the `lib` directory for the architecture.
- Use `InheritedWidget` to pass UseCase, Entity, and Store to the View.
- The diary entries will be accessible by date through a calendar. The calendar can be switched between monthly and weekly views. In the weekly view, the contents of the diary entries can be seen in a list format.
- Tapping a date on the calendar should display the diary below the calendar.
- Diaries can be edited and deleted. When deleting, show a confirmation dialog.
- Diaries can be created for today or any day in the past.
- Diaries should be saved in the `diary` table using `sqflite`.
- Perform Widget tests and Unit tests.
```
:::

# 結果
* 30日が全部今日の日付になっているカレンダー
* 真っ白の画面
* カレンダー部分が「CalendarView」となった仮置きのWidgetになっている

といった失敗作を経て5回目でまあまあ成功しました。

![アプリのスクリーンショット（カレンダー画面、日記記入画面）](/images/phb83junjy.webp)
![アプリのスクリーンショット（日記表示（月間、週間））](/images/r2nu4iushu.webp)

見た目も結構良い感じ。
作成は一回辺り1〜2時間くらいかかりました。AIの試行回数によって結構変動しそうですね。
価格はDeepSeekがセール中なので、まあまあ成功した版が入力37万5900トークン、出力9700トークンで$0.0231。158円換算で3.65円。
ただキャッシュが発生しているので価格の計算は難しそうです（間を置かずに連続で再作成した方が安くなりそう）
失敗した版が$0.0111〜$0.048と幅がありました。
まあまあ成功版を作るまでに掛かった費用は**合計$0.139**（**≒22円**）です。
DeepSeekのセールは主に出力が安くなっているので、セールが終わっても2倍程度の値上がりでしょうね。

コードは以下にあります。
https://github.com/hyshu/flutter_diary_app

ただし、まあまあ成功版にはいくつかの問題点がありました。

## 不具合
* 週間表示なのに「2 weeks」と表示される。この表示はボタンなのにタップしても反応しない
* 日記を書いても再起動したり、別の日付に移動すると消える
これは日記の日付が時分秒まで記録されてしまうため、日付をタップするとぴったり0時0分0秒に書いたのものしか表示されない状態になっていたためです。SQLのLIKE句を使用して簡単に修正できました。
* WidgetTestに失敗するのと、UnitTestが無い
テストは難しい処理のようで、8回も更新していました。最後のテスト項目が完成前にタスクが完了してしまったことが原因のようです。

## 気になった点
* 週単位に切り替えても一週間の日記が一覧で見れる訳ではない
プロンプトの「**週単位では日記の内容が一覧で見れます**」の「**週単位では**」が無視された結果、1日にいくつも日記が書け、同じ日の日記が一覧表示されるアプリになってしまったようです。
もっとプロンプトでの指示を具体的にする必要がありそうです。
* `InheritedWidget`が`StatefulWidget`の子ではない
`InheritedWidget`は状態を保持してくれる`StatefulWidget`が必要になります。あと`Builder`も欲しいところ。この辺りはAIが参考に出来るテンプレートを用意した方が良いのかもしれません。
* **勝手にパッケージをインストールされた**
追加したら駄目って言ったじゃん！
まあでも、テスト用のパッケージなので良いとしましょう…

# おわりに
昨日から始めたばかりなので改善点は多々ありそうですが、少し修正すれば済む程度のアプリが出力されたのは面白い体験でした。

テストも同時に作らせるのは難しそうなので、
**アプリを作成→アプリの仕様や設計、必要なテスト項目をAIに書かせる→それを元にテストを書いてもらう**
という流れが良さそうに思いました。

とはいえ、プロンプトのテクニックを磨いてもLLMの性能が上がったら不要になることが多いので、まずはClineの仕組みを理解することが大事なのかなと思います。
幸いコード量もそこまで多くなく、私でも何とか理解できそうなので、ちょっとずつ読んでいきます。