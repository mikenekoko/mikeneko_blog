---
title: Hexo と Netlify を使って簡単ブログをはじめよう！
date: 2018-11-24 19:36:33
tags:
- Hexo
- Netlify
- 無料ブログ
---
技術書店5というイベントでHexoとNetlifyという存在を知り、この2つを組み合わせたらいい感じのブログ環境が作れるんじゃないかな？と思ったので早速使ってみます！

## Hexoって？

* 静的なWebページ作成フレームワーク
  * markdownで書かれたファイルをHTMLに変換して出力する
* ブログに必要な機能を標準で備えている

ブログの中身はこちらのHexoを使うことで、静的コンテンツが出力されるのでWebサーバさえあればブログができます。

## Netlifyって？

* 静的サイト専用のホスティングサービス
* ssl等にも無料で対応
* Github等と連携し、自動でデプロイができる
* 無料で `100GB` も使える！

なんといっても、自動デプロイが便利です。
Githubと連携することで、 `git push` するだけでブログの記事が追加できちゃいます！

# 今回やりたいこと

1. Hexoで静的なHTMLを作成
2. それをGithubにpush
3. それをNetlifyで感知して、自動デプロイ&自動Webサイト更新
4. Slackに通知

![キャプチャ1.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/be4cbe7d-8f24-50cb-1acb-bd94c9016f7a.png)

# やってみよう！

## Hexo側の作業
最初は、Hexoを使ってブログを構築するところから始めます。

```
# 実行環境
Windowsで仮想環境を立ててます

Windows 10 -> virtualbox -> CentOS Linux release 7.5.1804 (Core)
npm 5.6.0
```
Centos上での作業になります。最初はHexoの環境を構築します。

```
$ sudo npm install -g hexo-cli
$ hexo init mikeneko_blog
$ cd mikeneko_blog
$ sudo npm install # hexoに必要とするものを一括インストール
```
生成された mikeneko_blogディレクトリがブログの中身になります。
`_config.yml` が設定ファイルなので、随時設定してください。デフォルトでももちろん動きます。

これで設定完了です！サーバーを起動して確認してみましょう。 `hexo server` を実行します。

```
$ hexo server
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```
表示された `http://localhost:4000` をブラウザで叩きます。

![キャプチャ.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/e5412c01-6285-e7c5-e519-9cca09525d96.png)

Hexoブログの雛形ができましたね！なんかもう最初からかっこいいです。すごい。

### 記事を作成する

記事を作成してみましょう！以下のコマンドで作成できます。

```
$ hexo new Hexo使ってみた！ # 任意の記事タイトル。日本語可
INFO  Created: ~/mikeneko_blog/source/_posts/Hexo使ってみた！.md
```

Markdownファイルが生成されました！こいつを編集して記事を作成します。
上部に見慣れない記述がありますが、記事の設定だと思ってください。以下の通り。

```Hexo使ってみた！.md
---
title: ブログのタイトル # 必須
subtitle: サブタイトル
description: ブログの説明 
author: 著者
language: 言語 # `ja` を記述しないと英語になる。
timezone: 時間 # `Asia/Tokyo` を書かないと仮想環境の時刻になる。環境によって左右されたくないので記述推奨。
tags: タグ # 複数指定する場合はymlっぽくハイフンで繋げます
id: URLの末尾を指定できる
---
ここから下にMarkdownで記事を書いていく
```

余談ですが、VSCodeのMarkdown Preview機能を使うといい感じで書けます。
書き終わったら、下記コマンドでHTMLファイル化します。

```
$ hexo generate
INFO  Generated: 2018/10/14/Hexo使ってみた！/index.html
```

今回作った記事の名前がついたhtmlが作成されているのがわかりますね。
サーバーを更新して、表示を確認してみます。

![キャプチャ１０.png](https://qiita-image-store.s3.amazonaws.com/0/178351/25ba274f-40e1-c004-b752-6f84f7fa8342.png)

先ほど作成した記事が確認できましたね！見た目もちゃんとしていてオシャレです。
これでHexoでのブログの作り方はわかったところで、次はこれをサーバーにアップロードできる仕組みを作ります。

## Netlify側の作業

最初は登録からです。

公式サイト (https://www.netlify.com/) にアクセスして、 `Get started for free` を選択。
連携したいアカウントを選択。今回はGithub連携を想定しているので、Githubを選んでください。
![1.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/6969ed05-3a8d-a88b-7003-76741cf9ddb3.png)
連携が完了したら、 `New site from Git` を選択。

![キャプチャ.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/256b7eed-bbda-c38f-98a3-ad30b0315839.png)
3ステップでいけるみたいですね。今回はGithubを選択。公開したいリポジトリを選択します。
![キャプチャ1.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/6ebf94d8-5aa2-887a-87af-deb5156b679a.png)
ここでGithub連携の自動デプロイ設定をします。上から順に解説すると、
### Branch to deploy
どのブランチをデプロイ対象とするか。今回は `master` にします。
これでmasterにマージすれば、デプロイが走ります。
### Build command
ビルドのコマンドを打ち込みます。
今回はHexoを使うので、 `hexo generate` を設定します。
### Publish directory
公開するべきディレクトリ。要するに、 `index.htmlのあるディレクトリ` 。
publicディレクトリ配下に作成したhtmlファイルが入っているので、 `public` を設定。

これで連携設定が終わりました！ https://happy-ride-4a84bd.netlify.com というURLが与えられましたね。最初からSSL化されててすごい。
ただ、サブドメインの謎の文字列が気持ち悪いので変えてみます。
`Settings -> Domain management -> Domains` に進み、右側の `…` マークから `Edit site name` を選択。
![キャプチャ4.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/01ba3d6e-099b-d84d-e821-215041f3c92d.png)
`https://(好きな名前).netlify.com` にする事ができます。
自分でドメインを買っている場合、それにすることもできるみたいですね。

# 確認してみる！

設定さえ終わってしまえば、こちらの作業はもう記事を書いて git push するだけで自動反映してくれるはずですね。さっそくやってみましょう！

```
$ hexo new 自動デプロイテスト
$ vi source/_posts/自動デプロイテスト.md # 記事の内容を書く
$ git add source/_posts/自動デプロイテスト.md # 記事の内容を書く
$ git commit -m "自動デプロイテスト"
$ git push origin master
```

先ほど実行した、mdファイルをhtmlファイルにしてくれる `hexo generate` はNetlify側で勝手にやってくれるので、やらなくても大丈夫です！
Netlify側で確認してみます。 `Production Deploys -> 先ほどコミットしたコミットメッセージを選択` と進めばデプロイログが表示されます。 
 
![キャプチャ5.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/192291d0-46b5-55ef-cd05-7283dc43716f.png)

先ほど追加したmdのhtmlファイルが作成されてますね！Webサイトを確認してみます。

![キャプチャ6.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/34f16ba3-9b1e-5f7f-6b50-796b01277878.png)

記事が反映されてますね！あと少しで設定終わりなので頑張ります。

## Slackにデプロイ完了を通知する
これもNetlifyで簡単に設定できます。 `setting -> Build&Deploy -> Add notification -> Slack Integration` と進んで設定するだけです。
![キャプチャ13.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/8f0693b3-7856-f625-c6d6-13882572cd10.png)

![キャプチャ7.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/57814d07-03dc-4b7f-9c9e-ce4163d1189c.png)

通知されましたね！シンプルでわかりやすいです。
通知と自動デプロイのおかげで、何か問題が起こった時以外はNetlifyの管理画面を開かなくても運用できるのが非常に便利です。

# 最後に
HexoとNetlifyを組み合わせたブログ環境、いかがでしたか？
一度環境を作って連携させれば git push するだけでいいのでメモレベルに気軽に記事を書けるのでオススメです。連携もシンプルにわかりやすいので是非使ってみてください！
ちなみに、このブログもHexoとNetlifyで組み立てました。テーマは調べれば有志で公開してくれている人がたくさんいるのでお借りしました。

それではまた。
