---
title: PerlフレームワークCatalystでCDN版のVue.jsを使った時にハマった話と解決方法
date: 2018-11-09 14:08:41
tags:
- perl
- catalyst
- vue.js
---
PerlフレームワークのCatalystのtemplatetoolkitファイルとVue.jsの記述はそのまま書くとハマります。

# 現象
```sample/report.tt
mounted () {                                             
  this.$nextTick(() => {                                
    const canvas = document.getElementById('daily_graph')
    this.chart = new Chart(canvas, {                     
      type: 'bar',                                       
      data: レポートデータ
      options: オプション
    })                                                   
  })                                                     
}                                                        
```

こんな感じでChart.jsを描画する処理を書いていました。オプションとかの書き方はChart.jsの公式リファレンス参照。
記述としては問題ないのですが、これをブラウザで実際に動かすと。。。

![キャプチャ4.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/da70b30b-38ee-8dd8-e949-e0821b68b0ce.png)

いやいや僕の書いてた `$nextTick` はどこ行ったんだよ。。 なんと消えてます。
調べても全く解決方法が出てこないので色々試したら解決しました

# 解決

\$マークの前にエスケープのバックスラッシュを追記します

```sample/report.tt
mounted () {                                             
-  this.$nextTick(() => {                                
+  this.\$nextTick(() => {  
    const canvas = document.getElementById('daily_graph')
    this.chart = new Chart(canvas, {                     
      type: 'bar',                                       
      data: レポートデータ
      options: オプション
    })                                                   
  })                                                     
}                                                        
```

![キャプチャ5.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/261c0875-dbfb-7943-20cb-6c294a40f88e.png)

表示されましたね！

## なんで？
正直わかんないです。ですが、Perlにとって\$マークはスカラー値の変数を意味するため$nextTickという変数を展開しようとして、そんなものはPerl側で記述してないので空になっていると思ってます（合ってるか知りません）

これ結構大問題で、$emitとかも全部エスケープする必要あります。早く.vueファイルに移行したいな～・・・ 
.ttファイルとの共存は難しそうです。

