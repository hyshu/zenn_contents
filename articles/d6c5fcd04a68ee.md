---
title: "Flutterエンジニアが Kotlin Multiplatform を試してみた"
emoji: "👌"
type: "tech"
topics: [ktolin, kmp]
published: false
---

# 今後の展望に期待する点

## ライブラリーの公開プラットフォームが欲しい
Flutterが人気である理由としてpub.devがあります。
本来はネイティブ側に手を出さなければならない実装も、パッケージがやっておいてくれるのでDartだけで記述できるようになることも多いです。

# KMPとFlutterは共存するのか
Kotlin Multiplatform が正式版になるということで「同じマルチプラットフォーム用フレームワークのFlutterはどうなるのか？」とネット上で話題になりました。
同時期にFlutterチームから解雇者が出たことに対し、Flutterチームのリーダーが言及した結果、「Flutterチームが解雇された」と勘違いした人が続出し、結構な騒動になりました。


KMPとFlutter両方を触ってみた感想としても、明らかに想定している開発者層が異なるため、最終的な判断はGoogle社次第ではあるものの、特に問題なく共存していきそうだなと感じました。

## 想定している開発規模が違う
**Kotlin Multiplatform**はアプリとWebのみならず、サーバー側もKotlinにすることを考慮に入れた設計になっています。
