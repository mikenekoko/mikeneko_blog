---
title: Vue Fes Japanに行ってきた！(後編)
date: 2018-11-05 16:32:37
tags: 
- vue.js
- vue-fes-japan
---

# note のフロントエンドを Nuxt.js で再構築した話

> 15:30 ~ 16:10 セッション4 Bホール

<iframe class="speakerdeck-iframe" frameborder="0" src="//speakerdeck.com/player/41b20d5bd3d44bd78352f7d07a8e231e?" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true" style="border: 0px; background: padding-box rgba(0, 0, 0, 0.1); margin: 0px; padding: 0px; border-radius: 6px; box-shadow: rgba(0, 0, 0, 0.2) 0px 5px 40px; width: 720px; height: 540px;"></iframe>

* noteはAngular.jsを使って2013年に開発開始したが様々な課題が生まれた
  * 初期表示速度が遅い
  * Rubyで書かれているので、Railsの上に載ったフロントエンド設計になってしまっている
  * 負債が多い。コンポーネントの設計など

`表示速度が!%下がるたびに、ECサイトだと離脱率が1%下がる` ！

そこでフルリニューアルを決意

## Vue.jsを選んだ理由
* コミュニティが活発であり将来性がある
* 触っている人が多い
* 実行速度と開発効率の両立をしている点
  * 差分描画などで速度もあり、コンポーネント概念が開発効率を上げている
* 学習コストの低さと親しみやすさ
* templateがHTMLと同じで書ける
* デザイナーでも直感で操作できる
* 公式ドキュメントの充実度

## Nuxt.jsを触るきっかけ
* `コーディング規約` が手に入る
* ディレクトリ構成や、ルーティングを自動で生成してくれる点が決め手。ルーティングをページごとに作っていくのは地獄

## 移行手順
* Railsサーバーは今まで通り使って、APIサーバーとする
* 目標：既存UIをページ単位で確実に移行させる。大幅なUI / UXの回収は含めない

## 苦労する点
* 並行して、現行版の開発はAngularで進んでいる = 2重メンテが必要

## 設計と実装
### 開発環境
* Nuxt.js
* Jest (テストツール)
* ESLint
* Angular.js版のコンポーネントを組み替えていく
* 今propsのバケツリレーが起きている状況。
  * これを解決するVuex + Atmic Design

## ファイルサイズの圧縮
* `nuxt build --analyze` とすることで、コンポーネントのデータ使用料を使うことができる

## Polifill.io
* UAごとに必要なPolyfillだけまとめて返してくれるもの。サーバー毎の最適化をしてくれるよ！
* nuxt.js.configで呼び出す。

## インフラ
* AWS Lambda
* EC2 + node + pm2

## まとめ
* パス(URL)ベースで小さく移行するのは有効！
  * 既存の言語やフレームワーク制約に縛られない
  * ロールバックが容易。簡単に戻せる
* デメリット
  * 完全移行まで地獄。並行して開発が進んだりする
* SSRの導入は簡単！
* 環境基盤のサポート規約が非常に強力！
* コストをかけずにモダンな開発環境が手に入る
* Routerなど、 `意識したくない部分は全部やってくれるから本来の開発に専念できる`

## 感想
* Nuxt、特に知らなかったがルーティングの自動化の恩恵が特に大きいと思った。
* NuxtはSSR必要なら有用らしいので組み込んでいきたい。

# 1年間単体テストを書き続けた現場から送る Vue Component のテスト

> 16:20 ~ 17:00 セッション5 Bホール

<iframe class="speakerdeck-iframe" frameborder="0" src="//speakerdeck.com/player/35bbd8b2a19645c1b2216f1d242fe691?" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true" style="border: 0px; background: padding-box rgba(0, 0, 0, 0.1); margin: 0px; padding: 0px; border-radius: 6px; box-shadow: rgba(0, 0, 0, 0.2) 0px 5px 40px; width: 720px; height: 540px;"></iframe>

## Componentのテストしてる人～！(挙手)
* 3割以下
  * バックエンドAPIサーバーとかで重要なところはテストしてるので、フロントはしてない人が多い

## 何をテストするの？（テスト観点）
* 外部からの見え方をテストする
  * `見た目に直で影響が及ぶ部分のテスト` を行うとよい
* input / output を整理

## どうやってテストするの？
* Props / VuexState テスト
  * 単純なassertテストかな？
  * 文言が変わっただけで、テストまで合わせる必要があり面倒
* `SnapshotTesting / VisualTesting`
  * componentのDOMの差分をスナップショットとして保存する
  * コード修正前のDOMを期待値として、テストする
  * GitHub上で、スナップショットの比較を行いいくつ間違えていたかも判定することができる
    * 不意にCSSが他ページに影響を及ぼすことがない！

## UIのテストは？
* 入力とか、クリックとか、スクロールとか…全部やるのは大変なので `入力 / クリック` くらいは簡単なのでそこだけテストしておきましょう
  * その他の部分に関しては、環境依存な点が多くテストする意味性が薄いです

## まとめ
* VisualTestは最高だからみんなやろう
  * 導入も簡単、レビューもデザイナーですらできるくらい簡単だよ！

## 感想
* スナップショットの画像比較テスト、フロントエンド界隈では基本なのかもしれないが非常に感動した。
* CSSが他ページに影響を及ぼしてしまい、修正したパターンがいくつもあるので今からでも実装したい。
* 導入は簡単らしいので、やってみます

# アフターパーティー

> 18:00 ~ 20:00

<img src="https://pbs.twimg.com/media/DrETu3SUcAASfTj.jpg">

豪華なご飯がたくさんありました！お寿司、ビーフシチュー、ビールにワインまで。。
7000円の参加費が安く感じますね(笑)
色々な方々とお話できてとても有意義な時間でした！
特に、Vueの生みの親であるEvan氏・Vueをやるきっかけになった本を出版されたmio氏にご挨拶できたのでとても満足しています…！

# 最後に
VueFesJapan2019も開催するらしいです！次回も絶対行こうと思えるとてもいいカンファレンスでした。
初心者の方も多く参戦しており、とても刺激を受けた、早く帰って開発がしたいとモチベーションが高い人が多かったです！
私もまだまだですが、次回までにはVueをバリバリ書けるようになりたいですね！

※一部画像は公式VueFesさんの画像URLを引用しています。
