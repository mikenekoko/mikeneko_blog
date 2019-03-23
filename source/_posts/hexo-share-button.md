---
title: hexoブログのシェアボタンを作ろう！
date: 2019-03-23 20:55:02
tags: hexo
---

Hexoのテーマによっては最初から共有ボタンがあったりするものの、色々とバグが多かったり単純にレイアウトが見づらかったりして私は好きじゃないです。
こういうのです↓
![1.png](https://qiita-image-store.s3.amazonaws.com/0/178351/71d174c4-074e-b281-a60d-6119936d732e.png)

私は記事毎にコメントフォームはいらないと思っていますし（そもそもコメント貰えない）、共有ボタンが小さすぎます。

ってことで、共有URLを作っていきましょう。TwitterとFacebookあれば十分だと思うのでその2つにします。

# 共有ボタンを作ろう！
## TwitterのURLを作る
形式は以下です。

```perl
# 正規の記述方法
https://twitter.com/intent/tweet?text=記事タイトル&url=記事URL

# 私のオススメの記述方法
https://twitter.com/intent/tweet?text=記事タイトル%20-%20ブログタイトル%0A記事URL
```

個人的に、記事タイトルの後ろにブログタイトルを付けたいので付与しました。 `%20` は空白記号のURLエスケープ表記です。
また、記事タイトルに改行記号の `%0A` を挟むのをオススメします。こうするとタイトルと記事URLの間が改行されて見栄えが良くなりますよ！

この記事だとしたら、URLは以下のようになります。

https://twitter.com/intent/tweet?text=hexoのシェアボタンを作ろう！%20-%20みけのエンジニアブログ%0Ahttps://mikeneko.tk/2019/03/23/hexo-share-button/

叩いてみると、よく見るこんな感じの画面になります！

![3.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/bd94676c-0744-3eb3-3610-396b2a013d45.png)

ではこれを目指して作成していきましょう。それぞれ以下で取れます。 hexoはnodejsのejsで動いているので、ヘルパーが使えます。

```perl
# 記事タイトル
<%= page.title%>

# ブログタイトル
<%= config.auther%>

# 記事URL
<%= page.permalink%>
```

これを組み合わせます。

```perl
https://twitter.com/intent/tweet?text=<%= page.title%>%20-%20<%= config.author%>%0A<%= page.permalink %>
```

## facebookのURLを作る
形式は以下です。

```perl
http://www.facebook.com/sharer.php?u=記事URL&t=記事タイトル
```

まぁfacebookで共有する人そこまでいないと思いますし、デフォルトにそって記述します。

```perl
http://www.facebook.com/sharer.php?u=<%= page.permalink %>&t=<%= page.title%>
```

# ボタンを作ろう！
こんな感じになります！WordPressのCSSを一部参考にしています。

<div class="share_link">
<a class="twitter_share_link" href="https://twitter.com/intent/tweet?text=hexoブログのシェアボタンを作ろう！%20-%20みけのエンジニアブログ%0Ahttp://mikeneko.tk/2019/03/23/hexo-share-button/" target="_blank">
  <i class="fab fa-twitter fa-3x"></i></a>
<a class="facebook_share_link" href="http://www.facebook.com/sharer.php?u=http://mikeneko.tk/2019/03/23/hexo-share-button/&t=hexoブログのシェアボタンを作ろう！" target="_blank">
  <i class="fab fa-facebook-f fa-3x"></i></a>
</div>

可愛いですね！ついシェアしたくなっちゃったらシェアしてもいいですよ！ｗ
ちょっと頭でっかちになっちゃってますが、hexoのコンパイル時の挙動でbrタグが入っちゃっています；；
正しく画面下部に埋め込んだら綺麗になるので、気にしないでください。

## HTMLとCSSを張り付ける

```html
<!-- アイコンを取得するCDNをheadに追加する -->
<link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.8.1/css/all.css" integrity="sha384-50oBUHEmvpQ+1lW4y57PTFmhCaXp0ML5d60M1M7uH2+nqUivzIebhndOJK28anvf" crossorigin="anonymous">

<a class="twitter_share_link" href="https://twitter.com/intent/tweet?text=<%= page.title%>%20-%20<%= config.author%>%0A<%= page.permalink %>" target="_blank">
  <i class="fab fa-twitter fa-3x"></i></a>
<a class="facebook_share_link" href="http://www.facebook.com/sharer.php?u=<%= page.permalink %>&t=<%= page.title%>" target="_blank">
  <i class="fab fa-facebook-f fa-3x"></i></a>
```

```css
/* シェアボタン */
.share_link {
    display: flex;
    margin-bottom: 30px;
}
.share_link a {
    transition: ease-in-out .2s;
    padding: 20px;
    flex-grow: 1;
    text-align: center;
}
.share_link a:hover {
    transform: scale(1.05);
    box-shadow: 1px 1px 4px 0px rgba(0,0,0,0.15);
    transition: ease-in-out .2s;
    background-color: #fff;
}
.twitter_share_link {
    background-color: #00B0ED;
    color: #fff;
}
.twitter_share_link:hover {
    color: #00B0ED;
}
.facebook_share_link {
    background-color: #3B5998;
    color: #fff;
}
.facebook_share_link:hover {
    color: #3B5998;
}
```

# 完成！
全ての記事に共有ボタンが適応されました！これでシェアしてくれる人が増えたら嬉しいな…！
やっぱりボタンがあると無いとじゃ人の行動って変わってくると思うので、是非設置してあげてください！

それではまた。