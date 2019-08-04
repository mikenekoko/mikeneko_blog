---
title: '[scala]ActorMaterializerのDIがうまくいかない'
date: 2019-08-05 01:34:45
tags: scala
---

# ActorMaterializerのDIがうまくいかない
ActorSystem や ActorMaterializer を毎回作成するのはとても処理が重く、スレッドを奪うためよくありません。

よくない例
```scala
class hoge {
  def fuga = {
    implicit val system = ActorSystem("on")
    implicit val materializer = ActorMaterializer()

    Http().singleRequest(
      HttpRequest(method = HttpMethods.GET, uri = "https://mikeneko.tk"
    )
  }
}
```

そこをレビューで指摘され、以下のように変形させました。
```scala
@singleton
class hoge @Inject() (implicit system: ActorSystem, materializer: ActorMaterializer) {
  def fuga = {
    Http().singleRequest(
      HttpRequest(method = HttpMethods.GET, uri = "https://mikeneko.tk"
    )
  }
}
```

だけど、以下のエラー
```perl
ProvisionException: Unable to provision, see the following errors:

No implementation for akka.stream.ActorMaterializer was bound.
```

ActorMaterializerのimplicitがない？？
IntelliJ上ではエラーにならず、同じ記述方法で書いている記事は複数ありました。
* https://blog.colinbreck.com/integrating-akka-streams-and-akka-actors-part-ii/
* https://www.programcreek.com/scala/akka.stream.ActorMaterializer

色々試してみてわかったのですが、ActorMaterialは `()` の宣言で挙動が変わってきそうです。
一回外に出して確認してみます。

```scala
// これは通る
@singleton
class Hoge @Inject() (implicit system: ActorSystem) {
  implicit val materializer = ActorMaterializer()
  def fuga = Try {
    Http().singleRequest(
      HttpRequest(method = HttpMethods.GET, uri = "https://mikeneko.tk"
    )
  }
}
```

```scala
// これは通らない(ActorMaterializerにかっこがないだけ)
@Singleton
class Hoge @Inject() (implicit system: ActorSystem) {
  implicit val materializer = ActorMaterializer
  def fuga = Try {
    Http().singleRequest(
      HttpRequest(method = HttpMethods.GET, uri = "https://mikeneko.tk"
    )
  }
}
```

結果
```scala
could not find implicit value for parameter materializer: 
akka.stream.ActorMaterializer
```

つまり `ActorMaterializer` はActorMaterializerとして判断されておらず、 `ActorMaterializer()` で判断されるようになるみたいです。

理由として、 `ActorMaterializer()` は applyが呼ばれるのですが、 `ActorMaterializer` だとただオブジェクトを呼んでいるだけっぽいです。
applyできずに、エラーになっているっぽいですね。ただのオブジェクトをimplicitとして渡されても困るって感じでしょうか。

```scala
object ActorMaterializer {

  def apply(materializerSettings: Option[ActorMaterializerSettings] = None, namePrefix: Option[String] = None)(implicit context: ActorRefFactory): ActorMaterializer = {
    val system = actorSystemOf(context)

    val settings = materializerSettings getOrElse ActorMaterializerSettings(system)
    apply(settings, namePrefix.getOrElse("flow"))(context)
  }

  ...
}
```

ActorSystemがカッコ無しで通るのは、抽象クラスになっていてそもそもの作りが全然違うからでした。

```scala
abstract class ActorSystem extends ActorRefFactory {

  ...
}
```

ならカッコ付きで宣言したらOK って思ったのですがここで新たな問題として `@Inject()` の後ろでimplicitとして定義したいのですがこの中で `ActorMaterializer()` という記述は使えません。カッコが使えないのです。（何故？）

ActorSystemはかっこなしとありで挙動が変わらなかったのでここに書いていますが、ActorMaterializerに関しては変わるので外に出すことにしました。やってることは同じなはず・・・

```scala
@singleton
class Hoge @Inject() (implicit system: ActorSystem) {
  implicit val materializer = ActorMaterializer()
  def fuga = Try {
    Http().singleRequest(
      HttpRequest(method = HttpMethods.GET, uri = "https://mikeneko.tk"
    )
  }
}
```

一旦これで。
毎回 ActorMaterializerを生成するのを防ぎたいという目標自体はクリアできたのでこれでOKとします。

これを解決するのに4時間くらい格闘してしまったので、誰かの助けになればなと思います。。