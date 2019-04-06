---
title: GitLabRunnerからDockerの操作がアクセス拒否される
date: 2019-04-07 02:42:34
tags:
- gitlab-ci
- Docker
---

Qiitaに投稿したものは調査段階の色々なものが混ざり合っているので、こちらをまとめた記事になります。
https://qiita.com/mikene_koko/items/27c9a4a0c8579a5b1de5

# 現象
GitLabCIのRunnerからDocker操作をするとログインしているにも関わらずアクセス拒否される

```java
$ docker login -u ******* -p ******* "$CI_REGISTRY"
Login Succeeded

$ docker build *********

$ docker tag ********* hoge/fuga/nyanyanya/scala:latest

$ docker push hoge/fuga/nyanyanya/scala

[info] Successfully tagged hoge/fuga/nyanyanya/scala:latest
[info] Built image hoge/fuga/nyanyanya/scala:latest
[info] The push refers to repository [hoge/fuga/nyanyanya/scala]
[info] 123456789abc: Preparing
[info] 123456789abc: Preparing
[info] 123456789abc: Preparing\
[info] 123456789abc: Waiting
[info] 123456789abc: Waiting
[info] 123456789abc: Waiting
[error] denied: access forbidden
```

ログインもできていて、ビルドもできていてpushするところでアクセス拒否されました。

ちなみに、ローカルだと問題なく動作できます。

```java
[info] Successfully tagged hoge/fuga/nyanyanya/scala:latest
[info] Built image hoge/fuga/nyanyanya/scala:latest
[info] The push refers to repository [hoge/fuga/nyanyanya/scala]
[info] 123456789abc: Preparing
[info] 123456789abc: Preparing
[info] 123456789abc: Preparing
[info] 123456789abc: Waiting
[info] 123456789abc: Waiting
[info] 123456789abc: Waiting
[info] 123456789abc: Pushed
[info] Published image hoge/fuga/nyanyanya/scala:latest
[success] Total time: 23 s, completed 2019/04/05 12:54:49
```

Runner経由の場合のみ動作しないので、実行環境の差による現象です。

# 結論：Dockerのバージョンが古い！
Dockerのバージョンが極端に古いとログイン処理で失敗してしまいます。

* ローカル環境 : `18.09.1`
* Runner環境 : `1.13.1`

もう全然バージョンが違いますね。原因は、Runnerで実行させているImageのDockerインストール手順でバージョンを一切指定していない事でした。

## Runnerサーバーでの実行環境用Dockerfile
```dockerfile
FROM centos:centos7

RUN yum update -y && \
    yum install -y wget tar java-1.8.0-openjdk java-1.8.0-openjdk-devel && \
    yum install -y docker && \
```

`yum install -y docker` としてるだけでバージョンを一切指定していませんでした！これでは古いDockerが入ってきてしまうのも納得です。最新のものを入れる記述に変更します。

```dockerfile
RUN yum install -y yum-utils device-mapper-persistent-data lvm2 && \
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo && \
    yum install -y docker-ce
```

これで `18.09.4` が入りました。執筆時点での最新です。これで再度実行してみます！

```java
[info] Successfully tagged hoge/fuga/nyanyanya/scala:latest
[info] Built image hoge/fuga/nyanyanya/scala:latest
[info] The push refers to repository [hoge/fuga/nyanyanya/scala]
[info] 123456789abc: Preparing
[info] 123456789abc: Preparing
[info] 123456789abc: Preparing
[info] 123456789abc: Waiting
[info] 123456789abc: Waiting
[info] 123456789abc: Waiting
[info] 123456789abc: Pushed
[info] Published image hoge/fuga/nyanyanya/scala:latest
[success] Total time: 36 s, completed Apr 5, 2019 9:43:14 PM
```

通りました！

ちなみに docker のイメージではなく centos7 のイメージを使ている理由としては、ほかにも色々インストールをしている事とcentos7は何度も使用している環境なので個人的にメンテナンスがしやすいからです。
何かプラグインの追加が必要になっても、yumで大体インストールできますしね！

# 最後に
Dockerのバージョンが古すぎると、Dockerログインしているにも関わらずアクセス拒否されてしまいます。
エラーメッセージからDockerのバージョンが古いのが原因というのが一切わからず、私は解決に2日かかったので誰かの解決の役に立てたら嬉しいです…！
エラーメッセージに出してくれたら助かったのですがｗ

今回の教訓として、CIでの動作バージョンは開発環境と合わせましょうという事ですね。あくまでローカルでやっていたビルド作業等をRunnerサーバーがやってくれているだけなので。
こういった例外が発生する元になります。

それではまた。