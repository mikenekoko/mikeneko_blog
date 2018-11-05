---
title: Vue Fes Japanに行ってきた！(中編)
date: 2018-11-05 16:12:39
tags: vue.js
---

# Vue Designer: デザインと実装の統合

> 13:50 ~ 14:30 セッション2 Aホール

<iframe src="//slides.com/ktsn/vue-fes-vue-designer/embed" width="700" height="540" scrolling="no" frameborder="0" webkitallowfullscreen="" mozallowfullscreen="" allowfullscreen=""></iframe>

## 今のWebアプリ / サイト開発について
* だいたい、フォトショとかでデザインして、JS/マークアップで実装みたいなことになっているんじゃないでしょうか？
  * 弊社も、Prottでデザインされたものの通りにHTML / CSSに落とし込んでいるので全く同じ
* デザインと実装で完全に役割が分かれている
  * デザイナーがフォトショとかでデザインを作り、エンジニアがその通りに実装

## デザインと実装が分かれることの問題って？
* １つ勘違いしないでほしいのは、わかれることのメリットもある！ということ。今回はデメリットの話

## デザインツールとコードは違う
* pngで書き起こされた画像を見ながら、コードで落とし込む。それはつまり2度手間になっている
* エンジニアが少しだけ機能追加をした場合、どうなる？いちいちデザイナーには報告しないだろう。すれ違いが発生する可能性がある

## ライブラリを使って解決策を考える
* [vuegg](https://vuegg.now.sh/)
  * プロトタイプでデザインしたものが、そのままVueのファイルで書きだしてくれるモノ
* [Framer X](https://framer.com/updates/)
  * vueeggでできない事ができる。ドラッグアンドドロップで画面を作成して、Reactのコンポーネントとして吐き出す
  * その吐き出したコンポーネントをFramerXに再度渡して修正もできる！（ここがvueeggと違う点）

## 作りたいもの
* SFC(.vueファイル)が実装、かつデザインできるもの
* 長期開発、運用に使える動的なデザインを可能なもの = FramerXに近いVue版
* レスポンシブにも対応させていきたい

その名も、 `Vue Designer`
<iframe src="https://hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fktsn%2Fvue-designer" title="ktsn/vue-designer" class="embed-card embed-webcard" scrolling="no" frameborder="0" style="display: block; width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;"></iframe>

## Vue Designerって？
* VueアプリのプレビューをVSCode上で見れるようにするプラグイン
  * プレビューを変更すると、自動でコードも変更される
  * デザインしたものがVueのコードになり、実装もデザインも同じコードになるためデザイナーでも使えるかも！？

## ロジック
* html
  * vue-eslint-parser
* js
  * @babel/parser
* css
  * postcss

## まとめ
* 将来的にデザインと実装が同じになればいいなと思い誠意制作中
* デザイナーも、エンジニアも同じツールで開発したいね！

## 感想
* Vueの最先端を走る人の作成しているプラグインの仕組みやデモ、どこに着眼したかの話を聞けた。
* 一人で作っているという事が信じられない程の完成度。歓声が楽しみ！
# Atomic Design のデザインと実装の狭間

> 14:40 ~ 15:20 セッション3 Aホール

<iframe class="speakerdeck-iframe" frameborder="0" src="//speakerdeck.com/player/6357deaee29c4b7c8c2a987d83f475d2?" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true" style="border: 0px; background: padding-box rgba(0, 0, 0, 0.1); margin: 0px; padding: 0px; border-radius: 6px; box-shadow: rgba(0, 0, 0, 0.2) 0px 5px 40px; width: 720px; height: 540px;"></iframe>

# デザイナーとエンジニアわかりあえてますか？

## コンポーネントベースにすることによる影響
* デザイナー
  * 習得の難易度が高い。
  * コンポーネントの設計ベースで開発しなければならない
  * なじみがなさすぎるので嫌がる
* エンジニア
  * 開発工数が少なくなる。開発にかかるコストの大幅軽減
* AtmicDesignであろうと、結局うまくいかない

## デザイナーが大事にしていること
* 効率化は大事だが、 `デザイナーが一番大事にしていることはユーザーの反応`
* だから、コンポーネントベースの話はとっつきにくい

## なんで流行らないの？
* そもそもコンポーネントはエンジニアリングの概念だから
* まだみんな慣れていない
  * これはかなり重要で、時間が解決する場合もある。今はオブジェクト指向が広まっているが、昔はC言語のような構造型言語がメインだった。なぜJavaのようなオブジェクト指向が広まったかというと、 `慣れと時間の解決` である
  * SketchはデザイナーにとってのJava
  * C言語はPhotoshop

## なんで慣れないの？
* 慣れない理由は、まだまだツールのサポートが足りてない、わかりづらいという点が大きい
* C言語しかやったことない人にJavaのフレームワークまで使ってアプリを作れと強いているようなもの
  * 個人的にこれめっちゃわかりやすいたとえで好きですｗ

## デザイナーはエンジニアじゃない
* デザイナーはユーザーにすげえ！って言われたい
* エンジニアは開発を楽にして工数を削減したい
* そもそも職責が違う

## まとめ
* この課題にベストアンサーはありません。デザイナーに求めるばかりでなく、エンジニアからサポートしてあげよう！
* デザイナーがデザインに専念できる環境を作ろう！

## 感想
* 自分はエンジニアで、デザイナーが作った画像の通りにコーディングするという典型的なここで上がっている対象だった。不満ばかり言っていたが、自分からできることもあるはずだと実感した。
* お互いに頼みあう関係でなく、協力しあえる関係にしたい！と強く感じました。
