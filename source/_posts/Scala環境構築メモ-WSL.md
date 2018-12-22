---
title: WSLのUbuntuにScala環境を構築する
date: 2018-12-22 18:55:21
tags:
- WSL
- Scala
---
みけです。なんと来年からScalaをやることになりました！
楽しみ半分、不安半分ですねw

とりあえず、環境を作っていきましょう！Scalaの環境に必要なものは以下です。

* JDK
* sbt

早速入れていきましょう！

# JDKをインストール
Java Development Kitの略です。Java本体ですね！
ScalaはJavaの延長線上と思っていただいていいので、こちらも必要になってきます。
今回はWSL環境（Ubuntu）で進めていきます。
まずはJavaコマンドがあるのか確認！

```
$ java -version

Command 'java' not found, but can be installed with:

sudo apt install default-jre
sudo apt install openjdk-11-jre-headless
sudo apt install openjdk-8-jre-headless
```

はい、入ってないですね。入れていきましょう。
まずはパッケージのインデックスを更新します。

```
$ sudo apt-get update
```

次にJDKのインストールを行います。
570MBとかそれなりにでかいので、容量には気を付けてください。。

```
$ sudo apt-get install default-jdk

...
163 MB のアーカイブを取得する必要があります。
この操作後に追加で 570 MB のディスク容量が消費されます。
続行しますか? [Y/n] Y
...
done.
```

確認

```
$ java -version

openjdk version "10.0.2" 2018-07-17
OpenJDK Runtime Environment (build 10.0.2+13-Ubuntu-1ubuntu0.18.04.4)
OpenJDK 64-Bit Server VM (build 10.0.2+13-Ubuntu-1ubuntu0.18.04.4, mixed mode)
```

```
$ javac -version

javac 10.0.2
```

# sbtのインストール
公式の手順を参考にしていきます。

```
echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
sudo apt-get update
sudo apt-get install sbt
```

何をしているかざっくり解説すると、現状sbtというパッケージがapt-getに認識されていないのでとりにいく記述とキーの設定をしています。

上から順番に実行していくと、2番目でエラー。

```
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823

Executing: /tmp/apt-key-gpghome.0pQOSILgXC/gpg.1.sh --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
gpg: connecting dirmngr at '/tmp/apt-key-gpghome.0pQOSILgXC/S.dirmngr' failed: IPC connect呼び出しに失敗しました
gpg: 鍵サーバからの受信に失敗しました: dirmngrがありません
```

鍵の追加でコケています。別にポート制限とかしてないんだけどなぁ…
ここで解決方法を数時間探ってかなり苦戦しました。

## 解決

こちらはR言語ですが、同じエラーでつまづいているIssueを発見。
https://github.com/Microsoft/WSL/issues/3286

この中のこちらを試してみます。

![キャプチャ1.PNG](https://qiita-image-store.s3.amazonaws.com/0/178351/a21f2347-25b4-ee3a-2b60-6927f8fc0343.png)

一度 `apt-get update` を実行してエラーにします。

```
$ sudo apt-get update

W: GPG エラー: https://dl.bintray.com/sbt/debian  Release: 公開鍵を利用できないため、以下の署名は検証できませんでした: NO_PUBKEY 99E82A75642AC823
E: リポジトリ https://dl.bintray.com/sbt/debian  Release は署名されていません。
N: このようなリポジトリから更新を安全に行うことができないので、デフォルトでは更新が無効になっています。
N: リポジトリの作成とユーザ設定の詳細は、apt-secure(8) man ページを参照してください。
```

ここで対象の値を取得します。今回は `99E82A75642AC823` ですね！
こちらを直接curlで鍵の追加をしてみます。

```
 $ curl -sL "http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x99E82A75642AC823" | sudo apt-key add

OK
```

OKが返ってきましたね！ `search=0x` の後ろに今回の対象を追加してください。
この後は引き続きコマンドを実行してください。

インストールが完了したら scala コマンドを実行できるか確認しましょう！

```
$ sbt console

Welcome to Scala 2.12.7 (OpenJDK 64-Bit Server VM, Java 10.0.2).
Type in expressions for evaluation. Or try :help.

scala>
```

scalaの対話型コンソールが立ち上がりましたね！

# 環境構築終わり！
ただ2つ、JDKとsbtをインストールしただけなのにめちゃくちゃ疲れました。
昔にwindows7本体に入れたときはchocolatyでサクっと入れれたのでむしろ苦戦しましたね。。ｗ

これでバリバリコードが書けますね！やっぱりWindowsは環境構築の難易度が高いな～と思うこの頃。
それではまた！