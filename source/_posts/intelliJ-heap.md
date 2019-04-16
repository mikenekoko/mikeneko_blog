---
title: [IntelliJ]sbtプロジェクトのヒープサイズの上げ方
date: 2019-04-17 00:52:20
tags:
- IntelliJ
- sbt
---

ターミナルから普通に動かせば動くのにIntelliJでBuildさせると動かない！
という現象に陥りました。エラーメッセージは以下の通り。

```javascript

Error while importing sbt project:

Error occurred during initialization of VM
Initial heap size set to a larger value than the maximum heap size
```

まぁメッセージ通りにヒープサイズが足りてないです。
これの面倒なところは2つ対処する必要があるところです。調べても１個目しか出てこない。

# 対処1. IntelliJのヒープサイズを上げる
こちらを参考にすればできると思います。
https://qiita.com/kazuki43zoo/items/49c90e5f05397c694d26

設定ファイルを作って、そこでヒープサイズを定義して底上げします。

```shell

$ vi ~/Library/Preferences/IntelliJIdea2016.2/idea.vmoptions

-Xms3048m
-Xmx3048m
```

これでIntelliJのヒープサイズが3GBになります。確かデフォルト700MBとかだったかな？

次に、sbtのヒープサイズも設定し直す必要があります。これはIntelliJ全体のヒープサイズを上げただけです。

# 対処2. sbtの実行ヒープサイズを上げる
こっちがなかなか調べても出てきませんでした。最終的に自力で調べてたどり着きました（ググるの下手くそ）

Preferences -> Build, Execution, Deployment -> Build Tools -> sbt と進んで下さい。

そこにある `Maximum heap size, MB` を変更します。 

<img width="1013" alt="スクリーンショット 2019-04-16 20.07.35.png" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/178351/6c50fc23-e00a-2279-76fe-14f10488513e.png">

私が見たときはデフォルト1500になっていましたが、 build.sbt に２G食うように記述していたのでここで失敗していました。

```java

  javaOptions in Universal ++= Seq(
    "-J-Xms2G",
    "-J-Xmx2G"
  ),
```

どれくらいあげればいいの？って感じですが、まぁ動けばいいかなって思ってるのでbuild.sbtに+1000MBくらいでいいんじゃないでしょうか。

設定し直して再Buildを実行して、ヒープサイズエラーが起きなかったら成功です。

IntelliJのヒープサイズを上げたとしても、SBTのヒープサイズというのが別に存在して動かなくなるなんて罠ですね…気をつけましょう。

迷ったら上げとけばいいと思います。