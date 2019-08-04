---
title: '[Scala]AkkaHttpでPostリクエスト'
date: 2019-08-05 01:58:14
tags: scala AkkaHttp
---

ScalaはRestAPIサーバーとしての役割にすることが多くて、Postするのは初めてで無駄に時間がかかったのでメモします。

```scala
import akka.actor.ActorSystem
import akka.http.scaladsl.Http
import akka.http.scaladsl.model.{HttpEntity, HttpMethods, HttpRequest, ContentTypes}

def postTest = {
  implicit val system = ActorSystem("on")

  Http().singleRequest(
    HttpRequest(
      method = HttpMethods.POST,
      uri = "http://example.com/hogehoge/fugafuga"
    )
    .withEntity(
      HttpEntity(
        ContentTypes.`application/json`,
        """{"test":"hoge"}"""
      )
    )
  )
}
```

## 解説
```scala
  implicit val system = ActorSystem("on")
```
* Akkaのお約束です

```scala
    HttpRequest(
      method = HttpMethods.POST,
      uri = "http://example.com/hogehoge/fugafuga"
    )
```
* HttpRequest: そのままリクエストを投げる構文です。Akka公式をみましょう
* method: POSTなのでPOSTです。AkkaのHttpMethodsを使います
* uri: POSTしたいURLをStringで定義

```scala
    .withEntity(
      HttpEntity(
        ContentTypes.`application/json`,
        """{"test":"hoge"}"""
      )
    )
```
* .withEntity: Entityを追加するときの構文です。PostはEntityを使うので、使用します。
* HttpEntity: Entity構文
* ContentTypes.\`application/json`: あまり見慣れない書き方ですが、中身を見ればこの書き方であっているのが理解できます。(下記に記述）
* """{"test":"hoge"}""": Postしたい物をStringで記述します。ContentTypeと合わしてください

```scala
object ContentTypes {
  val `application/json` = ContentType(MediaTypes.`application/json`)
  val `application/octet-stream` = ContentType(MediaTypes.`application/octet-stream`)
  val `text/plain(UTF-8)` = MediaTypes.`text/plain` withCharset HttpCharsets.`UTF-8`
  val `text/html(UTF-8)` = MediaTypes.`text/html` withCharset HttpCharsets.`UTF-8`
  val `text/xml(UTF-8)` = MediaTypes.`text/xml` withCharset HttpCharsets.`UTF-8`
  val `text/csv(UTF-8)` = MediaTypes.`text/csv` withCharset HttpCharsets.`UTF-8`

  // used for explicitly suppressing the rendering of Content-Type headers on requests and responses
  val NoContentType = ContentType(MediaTypes.NoMediaType)
}
```

## 詰まったポイント
##### ContentTypesは必ず akka.http.scaladsl の物を使う

* akka.http.javadls とか、 play.api.http のやつを使うと一生エラーになる

##### HttpEntityの第二引数はString

* Jsonだし、JsValueで渡そう〜と思ったらStringしか受け付けないみたい。シリアライズする時とかに型を意識したほうがいいかも
* JsValueとかで渡してしまった場合は、 `toString` で対応しよう（動くかはわかりません。。）

##### HttpRequestのkeyはしっかり書く

* `method = HttpMethods.POST` を、単に `POST` として書いているコードがgithubに多かったが、僕の環境が悪かったのか動かなかったが定義して書いてやると動いた。
* 同じ悩みを抱えている人は一回このように書いてみて試してみてもいいかも

##### .wiithEntity(HttpEntity(~~~)) の順番 

* あんまり使わないので、ごっちゃにならないように注意