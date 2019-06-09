---
title: Vue Fes Japanに行ってきた！(前編)
date: 2018-11-04 20:44:37
tags:
- vue.js
- vue-fes-japan
---
<img src="https://www.yumemi.co.jp/application/files/1815/3803/8559/vuefesjapan2015.png">
お疲れ様です、みけです。
本日はVueFesJapanにやってきました！その様子を開場から追っていきたいと思います！


## タイムテーブル
[こちら](https://vuefes.jp/time-table/)に書いてある通りです。
<img src="https://qiita-image-store.s3.amazonaws.com/0/178351/4c9e046c-bb40-b9a0-63c6-3126c83c4ba0.png">

# 開場
> 9:30 ~ 10:20

<img src="https://pbs.twimg.com/media/DrCh5uTUUAEh7fA.jpg">
既に入り口付近がすごい人だかりです！
様々な企業がブースを構えています！
有名な企業だと、LINEやMicrosoftさんがいました。

---

# keynotesession
> 10:30 ~ 11:20

<img src="https://pbs.twimg.com/media/DrCuHY4V4AE0bob.jpg">
Vue.jsの生みの親であるEvan氏による、Vue.js3.0リリースの話と今までのVue.jsのお話。
英語で話されているのを、スタッフが同時翻訳するというスタイル。
詳しく書いていきます。

# Vue.js 3.0 リリースの話
* より早く
* より小さく
* よりメンテナンスしやすく
* よりネイティブ向けに作りやすく
* よりあなたのコードの保守性を向上

主な変更点は上記5つであると話されていました。

## より早く
### 仮想DOMの実装を作り直して最大化

* mountとpatch処理が最大100%に向上する

### プロキシを用いた監視の強化

* 今後は、クラスといったものも監視できるようになる

### コンパイルの仕組みを高速化

* 実行中のオーバーヘッド削減のためコンパイル時にヒントを追加(地味に嬉しい)
* コンポーネント探索の高速化
  * コンポーネントの状態をテンプレートに認識させ、子要素が存在しない場合は中身を探索しない等、無駄な処理を省き高速化を実現
* 静的ツリーを巻き上げる
  * テンプレートを静的部分と動的部分で区別して、処理を分離させる
  * Laizy Fanctionという仕組みを取り入れることにより、子要素は必要な場合のみ呼び出されることになる

### `単純に、3.0にアップデートするだけで早くなる`

* たとえば、表示に284msかかるサイトがあるとすれば、Vue3.0に上げるだけで164msにまで向上します！
* Vue2.5から3.0にすることで、メインメモリ使用量も速度も2倍レベルになる
* 特に書き換えることなく上記の最適化が行われ速度向上が見込めます！

すごいですね、Vue3.0で登場した新機能の書き方に変えれば～というわけでなくアップデートするだけで速度が向上するなんて。感激です。

## より小さく
### Tree Shakingへの対応

* 使われていないコードをバンドルされたファイルに含まないようにする機能
  * 様々な分野で使われていない部分を感知する
    * ビルドインのコンポーネント(keep-alive,transition)
    * ディレクティブのヘルパー (v-model,v-for）
    * ユティリティ関数(osyncComponent,mixins)

Tree Shakingについては[こちら](https://qiita.com/pirosikick/items/863830856891d40308cb)

### 新コアのランタイムサイズは、Gzipで10kb以下になる。

* Vueはより軽いものになる！

## よりメンテナンスしやすく

### アーキテクチャの整理

* アーキテクチャの整理を行った関係で、Javascriptが動く華僑があればどこでも実行することができます
  * それはつまり、ブラウザーがなくても開発できる環境を意味しています！

### パッケージの分離

* 状態を変更した場合、それは複数のコンポーネントに影響を与えます
* この場合にどのコンポーネントにそれを適応させたらいいのか考えてくれます。

### テストセットアップの改善


## よりネイティブ向けに作りやすく

### カスタムレンダラAPI

* カスタムレンダラAPIを使うことで、ネイティブ向けのアプリを使いやすくなります
* これは、ネイティブ向けの開発者ツールになります。

## よりあなたのコードの保守性を向上

### リアクティビティAPI

* リアクティビティAPIを使うことで、どんなコンポーネントでも監視できるようになる
  * observableメソッドでVuexのstateの変更を監視できる
  * storeを変更することですべての参照している箇所をアップデートできることができます
  * Evan「今後、Vuexは必要ないかもしれませんね」と言っていた
* リアクティブの画面に反映

### コンポーネントの再描画を理解しやすくする

* Render Trackという仕組みを導入
* デバッグをしやすくなる

### TSXによるTypeScriptサポートの強化

* TSXでRender関数を記述することができる
  * Vue2のTSのサポートはそんなにいいものではなかった

### HooksAPI

* Reactで登場した機能を導入
* コンポーネントのロジックを梱包するのが簡単になる
* mixinのネームスペースが被る問題の解決策になる
  * たとえば、5個のmixinがあった場合、どこを参照しているか非常にわかりづらい
  * Hooksを使うことで、簡単になります。管理もしやすい！
    * APIからXML形式で結果が返ってきます
    * それをMutatiosnで受け取り、3つの率い数で受け取ってそれをrender関数wに渡すだけで良くなるのです

### Time Slicing

* 重い処理があったとしても、ブロッキングしなくなる。処理を分断させる
* たとえば入力フォームに入れた内容を、バインドしてコンポーネントで読み込み表示するとします
* 入力されるたびにコンポーネントが更新され、おもくなります
* 高速でタイピングしたとして、hoge と売った場合、hog は描画するものの、最後のeが描画された瞬間に最初の3文字を描画する処理は必要なくなります。これをslicingは自動で判断し、処理をスキップします
* これにより、パフォーマンスの大幅向上が見込めるのです

処理の実行中に新しい処理のキューが溜まった場合、古い処理を捨て新しい処理に写るというもの。非常に便利そうだ。

---

# ランチブレーク

> 11:30 ~ 13:00

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">Vue Fes Japan 2018のランチセッションの弁当は人形町今半のすき焼弁当だった。 <a href="https://twitter.com/hashtag/vuefes?src=hash&amp;ref_src=twsrc%5Etfw">#vuefes</a> <a href="https://t.co/2oA60XqCeJ">pic.twitter.com/2oA60XqCeJ</a></p>&mdash; iwbjp (@iwbjp) <a href="https://twitter.com/iwbjp/status/1058548833938853888?ref_src=twsrc%5Etfw">2018年11月3日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

本来は、ここでランチを食べながらスポンサーのセッションを聞くことができるのだが先着から外れてしまったので惜しくも外で一人で食べることに。
内容が気になる方は他の方のVueFesJapan2018のまとめ記事をどうぞ。
すき焼きおいしそうだなぁ。。。

---

# Vue.js と WebComponents のこれから

> 13:00 ~ 13:40 セッション1 Bホール

<iframe class="speakerdeck-iframe" frameborder="0" src="//speakerdeck.com/player/0bf75fbec2584e69a35b067b43ddee41?" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true" style="border: 0px; background: padding-box rgba(0, 0, 0, 0.1); margin: 0px; padding: 0px; border-radius: 6px; box-shadow: rgba(0, 0, 0, 0.2) 0px 5px 40px; width: 700px; height: 540px;"></iframe>

* https://vuefes.jp/speakers/takanorip/
* 本出版中：イヌでもわかる web components

# WebComponentsとは？
* Web標準でカプセル化されたコンポーネントを作る技術
* 独自のHTMLタグを作成するためのWebプラットフォームのAPI
* 詳しくは[こちら](https://qiita.com/jtakiguchi/items/b1315f53b3726ff11b61)

## WebComponentsの4つの仕様
### Custom Elements
* 独自の機能や見た目を持ったHTML要素を定義できるようにするための仕様
* つまり、自分でHTMLのタグを作ることができる

### Shadow DOM
* DOMのカプセル化を行う機能
* Vueのcomponentの元となったような仕様で、CSSやJSがscopedに作ることができる

### HTML template
* `<template>` 要素を使用して独立したHTMLを形成できる

### ES Modules
* ES5で導入されたES Moduleの機能を利用して、外部で定義されたカスタム要素を読み込む(JSファイル)
* 残念ながら、 `2019春` にGoogleChromeから削除される予定

### HTML Modules
* HTMLをJSの中に直接Importできるようする仕様
* 同時にCSSModulesも開発されてるよ！
* またIssueで話されてるだけの段階らしい

## で、どうやってWebComponentをVue.jsで作るの？
* VueCLI3の機能である、 `Build Targets` を使うと、WebコンポーネントをVueで使うことができる！
* VueCLI3でbuild するときに、 `--target wc` というオプションを付けると、Vue.jsのコンポーネントをWeb Componentsに変換して生成される
  * これにより作成されたファイルは、単体でWebComponentsとして動かすことができる
  * つまり、Vueから簡単に移行することができる
  * 部分的な導入も簡単です！

## なんでWebコンポーネントを使うのがいいのか？
* `UIフレームワークを統一できる` ！ここが一番大きい
  * フレームワークを変更しなくてはならなくなった場合に、UI部分はを使いまわせる
* `完全にスコープである` ！
  * WebComponentsでは、Shadowを使用することでスコープな実装が可能となる。JSは完全なスコープ設計ではないので、そこをカバーできる
  * たとえば大プロジェクトでCSSの依存関係が意味不明になっていた場合に活躍できる
* デメリットとしては、CSSの設計を大幅に見直す必要があること、外部から渡されるイベントハンドリングが面倒なこと

## 正直なところは？
* 現状、Vue.jsの機能を使ってコンポーネントを作ったほうが簡素で簡単

## じゃあなんでやるの？

### Micro Frontend化させていきたいため
* 大きなサービスや構造、ページは細かく分けてお互いに影響を及ぼさないような設計にしようという考え方
  * たとえば、CSS変えるとかライブラリを変えたり、JSを変えたりとか。簡単に変えるとアプリケーション全体が壊れてしまうということが起こりえる
  * ここでWebComponentsならScopedなので変更が容易！
* Vueもいつか死ぬ。 `フレームワークを使っている時点で技術負債を作っているという自覚を持とう`
  * そこで、WebComponentsを使うことで簡単に移行することができる。
  * これがWeb標準である強み。 `技術負債を貯めない` ということに重点を置くならば、WebComponentsは有用。

## 将来どうなるの？

### Vue.jsは将来的にWebComponentsに置き換えられていくの？
* そんなことはない、どちらかというと共通して生きていくことになる
* Vue.jsはデータバインドとか、データのやり取りで有効活用していき、WebComponentsは末端に行ってほしい
* WebComponentsとはHTMLをカプセル化させる目的で使われるでしょう
  * HTMLなどの干渉をなくしていこうという考え方です
  * Vue.jsはウェブアプリケーションを作るためのフレームワーク。そこは、Vue.jsでやっていきましょう

## まとめ
* WebComponentsは標準搭載されていてもう使うことができるが課題も多い
* Vue.jsとWebComponentsは共存できます！
* UIをWeb Componentsに任せることで、負債を減らすことができる
* アプリケーションサイドはVue.jsに任せることがいい

`Let's use Web Components!`

## 感想
Webコンポーネントという存在すら知らなかったが、非常に有用な事が伝わった。
特に、ShadowDOMによるscopedな開発環境はCSS干渉を起こしてリリースし直したり影響範囲を調べたりといったしがらみから解放されるのは魅力的！

中編の記事は[こちら](/2018/11/05/Vue-Fes-Japanに行ってきた！2/)
