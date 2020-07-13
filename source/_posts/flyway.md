---
title: "[Scala] Flywayでテストコード用のDBを統括管理する"
date: 2020-07-13 20:19:12
tags:
---

# Flywayとは？ 
こちらの記事がわかりやすいです。
https://qiita.com/osuo/items/3aa375f1a1d6dd3d2459
https://qiita.com/opengl-8080/items/6368c19a06521b65a655

同じ状態のDBを再現できるDBマイグレーションツールです。

# なぜテストデータのDBとして扱うことを決めたか
テストデータの作成 / 管理がしんどくなったからです。また、ありえないデータが作られていてテストの担保が怪しくなっていました。

今のテストデータ入力はこんな感じ。

```scala
val testQuery = 
  sqlu"""
    INSERT INTO
      `user` (`id`, `name`, `sex`, `age`)
    VALUES (
      '1', 'tanaka_taro', '0', '1',
      '2', 'yamada_hanako', '0', '1',
      '3', 'saito_matsuko', '0', '1',
    )
  """

  Await.result(db.run(testQuery), Duration.Inf)
```

年齢が全て1歳で登録されています。この後に続くテストコードでは、年齢に関するテストをしない予定だからです。
性別も、男女関係なく0に設定しています。0は男です。

次に、年齢が１８歳以上と未満で分岐するメソッドのテストコードでのデータを見てみます。

```scala
val truncateQuery =
  sqlu"""
    TRUNCATE TABLE user
  """
  Await.result(db.run(truncateQuery), Duration.Inf)

val testQuery2 = 
  sqlu"""
    INSERT INTO `user` (`id`, `name`, `sex`, `age`)
    VALUES ('1', 'test_user1', '0', '19'),
           ('2', 'test_user2', '0', '18'),
           ('3', 'test_user3', '0', '17')
  """

  Await.result(db.run(testQuery2), Duration.Inf)
```

これは一度userテーブルを消してます。前の年齢がおかしいテストデータが入っているからですね。
消した後に、年齢が正しいデータを入れています。また、今回は名前に対してのテストをしないので名前が `test_user1` みたいにありえない値になっています。

このように、実行するテストと感心の無いデータは極力適当な値を入れることで何のテストをしたいデータなのかを明確にするようにしていました。ただ、その結果データが本来ありえないものになっていますし、毎回データのinsertを見る / 書くが必要になってきてとにかくしんどくなってきました。

ってことで辛いので、Flyway1つにデータが集約されるように頑張ってみます。

# 導入してみる

```build.sbt
libraryDependencies ++= Seq(
  ...
  "org.flywaydb" % "flyway-core" % "(使いたいバージョン)" % Test
)
```

テスト用なので、テストフォルダにSQLを追記していきます。
階層は assets で追加します。構造などは公式を参考にしました。
マイグレーションファイルは、 `V(バージョン)__(説明).sql` 形式で配置します。

空のデータベースが用意されているだけの状態なので、初期テーブルを定義する base を定義しその後データを追加する構文を追加していくイメージ。

```
test
├── domain
├── infrastructure
├── controllers
└── assets
    ├── db
    │   └── migration
    │       ├── 00__base （全テーブル作成）
    │       │   └── V1__initial.sql
    │       ├── 10__user (userテーブル)
    │       │   └── V1__add_user.sql

            ...（都度テーブルごとに増やしていく）
```

migration/base/create_db.sql

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL,
  `sex` int(11) NOT NULL COMMENT '0:男 1:女 2:未回答',
  `age` int(11) NOT NULL,
  PRIMARY KEY (`id`),
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

migration/user/create_data.sql

```sql
INSERT INTO `user` (`id`, `name`, `sex`, `age`)
    VALUES ('1', 'tanaka_taro', '0', '19'),
           ('2', 'yamada_hanako', '1', '18'),
           ('3', 'saito_matsuko', '2', '17');
```

呼び出し用のObjectを作成します。

```app/infrastructure/mysql/Migrate.scala
package ...
import ...

object Migrate extends Databases {
  private val flywayBase: Flyway = Flyway.configure().dataSource(
    ("dbのURL"),
    ("dbのUSER"),
    ("dbのPASSWORD")
  ).locations("classpath:db/migration/00__base").load()

  private val flywayUser: Flyway = Flyway.configure().dataSource(
    ("dbのURL"),
    ("dbのUSER"),
    ("dbのPASSWORD")
  ).locations("classpath:db/migration/10__user").load()

  def base(): Unit = {
    flywayBase.clean()   // 初期化
    flywayBase.migrate() // マイグレーション実行
  }

  def user(): Unit = {
    flywayUser.migrate() // マイグレーション実行
  }
}
```
これで、 Migrate.base() でテーブルをクリーン状態に、 Migrate.user() でユーザーのデータを追加ができるようになりました。

呼び出してみます。

```scala
import org.scalatest.{BeforeAndAfterAll, FreeSpec, Matchers}

class MySQLUserSpec extends FreeSpec with Matchers with BeforeAndAfterAll with Databases {

  override def beforeAll(): Unit = {
    Migrate.base()
    Migrate.user()
  }

  "MySQLUserSpec" - {
    "Userテーブルに関連するテスト" - {
      "完全一致する名前のユーザーの情報を取得できる" in {
        MySQLUserRepository.findUserByUserName("tanaka_taro") should be (田中太郎のデータ)
      }
      "18際以上のユーザーの情報を取得できる" in {
        MySQLUserRepository.findAdultUser should be (田中太郎と山田花子のデータ)
      }
    }
```

`beforeAll()` でflywayのマイグレーションを実行してDBを今回使いたい状態にします。

このテストで実行したいのは、userテーブルなので `Migrate.user()` を追記します。
他のテーブルも必要な場合は、随時migrationファイルを記述して追記していきましょう。
`Migrate.base()` の読み込みも忘れずに記述します。

### もう少し拡張してみる
userデータをデフォルトで何件か登録して置きたいとして、毎回userを呼び出すのはしんどいのでベースのデータとして登録するように設定します。
`V2__add_base_data.sql` を追加します

```
    ├── db
    │   └── migration
    │       ├── 00__base （全テーブル作成）
    │       │   └── V1__initial.sql
    │       │   └── V2__add_base.sql
    │       ├── 10__user (userテーブル)
    │       │   └── V1__add_user.sql
```

V2__add_base.sql

* V1__add_user.sql にを移動させてあげます。

```sql

INSERT INTO `user` (`id`, `name`, `sex`, `age`)
    VALUES ('1', 'tanaka_taro', '0', '19'),
           ('2', 'yamada_hanako', '1', '18'),
           ('3', 'saito_matsuko', '2', '17');
```

デフォルトで必要なデータはここにどんどん記述していく形ですね。例えば、ユーザーの購入履歴を調べる場合などにユーザーテーブルは必ず必要で、その度に作成文を呼ぶのはあまり効率的ではありません。
必ず必要になる、ベースとなる根幹のデータはここに追記していきます。

その場合、 `V1__add_user.sql` にはそのテストで必要なパターンのみ追記するなどがいいと思います。

70歳を超えられている方には割引キャンペーンを適応させるロジックがあるとして、それは一部でしか使われない機能で全体として持つ必要がないデータだとします。
そのデータをbaseとして共通で持たせてしまうと、baseが汚れてしまうのでそういった一部のニーズのデータは個別にinsertする文を追加してあげます。

V1__add_user.sql

```sql
INSERT INTO `user` (`id`, `name`, `sex`, `age`)
    VALUES ('100', 'hayashi_jirou', '0', '74'),
           ('101', 'kobayasi_sachiko', '1', '70'),
           ('102', 'hasegawa_takahiro', '0', '78');
```

IDはこのSQLで追加で登録したユーザーだとわかるように100番台にしています。ここはルールをチームで決めればいいと思います。

既存のbaseのデータに追加したい場合は、flywayの履歴管理テーブルを一度消すことで実現可能です。

flywayの 3系、4系では `schema_version ` 、5系では `flyway_schema_history` という名前です。

```scala
  def user(): Unit = {
    Await.result(db.run(sqlu"""delete from flyway_schema_history"""), Duration.Inf)
    flywayUser.migrate()
  }
```

### 導入完了してみた感想
DBの状態がflywayで統括されるようになった上にコードがとてもスッキリしたので満足しています。

これを実行するだけで

```scala
    flywayBase.clean()
    flywayBase.migrate()
```

DBがあるべき状態に戻せるようになったので異常なデータが入っていない事を担保できるためテストの信頼度も上がりますね！

追加で何か必要な場合、別途SQLを書けばいいのでそこだけ見れば「基本の状態にこれを加えた状態だ」とわかるので今後その追加構文だけ見ればDBの状態も理解しやすいです。

色んなテストコードにバラバラでSQL追加文を書いていましたが、これからflywayで統一できるので可読性が一気に上がりました。同じ悩みを抱えている方がいましたらオススメです。
