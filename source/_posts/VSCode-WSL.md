---
title: VSCodeでWSL環境と接続する
date: 2018-12-16 11:01:22
tags:
- WSL
- VSCode
---
どうもみけです！WSLの設定楽しすぎて連続での記事投稿になります。
メインではVimを使ってるんですけど、VueやScalaを書く事が今後多くなりそうなのでエディタをこの機会にVSCodeに変えてみます。
とにもかくにも接続設定が必要ですね！

# 今回やること
* VSCodeのターミナルからUbuntuを呼び出すようにする
  * フォントを変える
* WindowsとVSCodeでディレクトリを共有する

# VSCodeのターミナルからUbuntuを呼び出すようにする
さぁやっていきましょう！VSCodeのターミナルは `Ctrl+@` で呼び出せます。
現状だとWindowsのデフォルト環境を開いちゃってるんで、Ubuntuを呼び出すようにしましょう。

![キャプチャ1.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/300e3387-90c2-ff44-1d1f-740df30f2d5d.png)

`ファイル->基本設定->設定` と進んで検索ボックスに下記を入力します。

```
terminal.integrated.shell.windows
```

そこから、ターミナルのパスを通します。自分の環境の場合は下記です。
環境ごとに合わせましょう！

```
C:\WINDOWS\System32\bash.exe
```

一旦設定終わったら再起動して、ターミナルを起動してみましょう。

![キャプチャ2.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/89e3a239-6f4b-71c8-9971-cd65b14b2dc7.png)

文字化けしとる。。。

## フォントを変える
![キャプチャ3.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/afa74d42-accd-e2de-1992-0387f730c754.png)

適当に検索で `terminal font` とかで検索したらヒットしました。絶対こいつですね。
調べてみたら同じissueがありましたhttps://github.com/Microsoft/vscode/issues/7116
同じように設定します。

```
Source Code Pro for Powerline
```

を設定して、再起動。

![キャプチャ4.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/59a2cbb0-162c-e4d4-0102-a7595188a146.png)

フォントがちゃんと予想通りになりましたね！かっけぇ…これで万事OKです。

# WindowsとVSCodeでディレクトリを共有する
これができれば今までSambaとかで必死に共有ディレクトリとか作ってたのがアホらしくなりますね。
昔からずっとこれができずにMACにあこがれてたので是非とも成功させたいところ。
といっても、 `/mnt/c/` ディレクトリ以下がもう共有されているのでここ使えばいいだけです。
WSLすげー！！

![キャプチャ5.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/f7cfc7eb-e38c-3745-73a8-8e9cbde8eba1.png)

VSCodeのディレクトリ見てほしいんですけど、デフォルトでもう `/mnt/c/User/あなたの名前` ディレクトリにいるんです。
賢すぎか？つまり、ここの下にディレクトリ使えばもう共有ディレクトリになります。
今何もないので、なんかディレクトリを作って見ましょう。

```
mkdir test
```

windows側で自分のフォルダを見てみます。 testディレクトリがあればOKです！

これでどこに共有フォルダができるかわかったので、こちらのディレクトリをVSCode側で呼んであげればOKですね！

## 注意
共有されたとはいえ、Ubuntu側で作成したファイルをWindows側で直接いじるのは厳禁です。ファイルが壊れます。
こちらから引用。https://qiita.com/kaitoy/items/e20d426cdd1f507bfddb

> 簡単に言うと、Windows側のプロセスがUbuntu側のファイルを作ったり編集したりする際、パーミッションなどのメタデータを適切に設定しないため、
> Ubuntu側でファイルが壊れたと判断されてしまうから。

やはりWindowsはクソなのか。。。？

# 設定完了！
![キャプチャ6.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/cbcd2ea2-fa20-59ea-9d3d-2b08dc2e8369.png)


いやぁVSCodeだけでブログの投稿が完結できる環境が立てれました。便利ですね！
markdownで書いて、プレビューを見て、下でコンソールを叩ける。完璧すぎる！

VSCodeをVimみたいに使うプラグインはいっぱいありますがAtomを似たような設定にして挫折した過去があるのでまたの機会に。
一旦デフォルトで使うのも悪くないですしね！しばらくはこのVSCodeライフを満喫しようかな～と思います！
それではまた。
