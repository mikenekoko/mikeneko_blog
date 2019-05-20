---
title: '[Nginx]worker_connectionsとworker_rlimit_nofileの値は何がいいのか？'
date: 2019-05-20 10:11:53
tags:
- nginx
- worker_connections
- worker_rlimit_nofile
---

worker_connections を上げたいけど、どの程度上げていいのかわからない！
って事で、どこをみてどの値を設定するのがベストなのかいろいろ調べて調査したのでその調査記録になります。

# 現象
弊社のサービスで以下のNginxエラーが出てしまいました。

```perl
2048 worker_connections are not enough
```

worker_connectionsが足りなくなってしまっています。
今回、ここの値を上げるに当たってworker_rlimit_nofileも上げる必要があります。

# worker_rlimit_nofileの値は何にすべき？
## そもそもworker_rlimit_nofileとは？
Nginxのworker_connectionsの1workerプロセスにおける、 `ファイルディスクリプタ` の上限値

引用） [Nginx - worker_rlimit_nofile](http://nginx.org/en/docs/ngx_core_module.html#worker_rlimit_nofile)
> Changes the limit on the maximum number of open files (RLIMIT_NOFILE) for worker processes.
> 和訳） workerプロセスの最大オープンファイル数の上限（RLIMIT_NOFILE）を変更します。

## ファイルディスクリプタとは？

引用） [IT用語辞典](http://e-words.jp/w/%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%83%87%E3%82%A3%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%97%E3%82%BF.html)
> ファイルディスクリプタとは、プログラムがアクセスするファイルや標準入出力などをOSが識別するために用いる識別子。

佐々木真先生の図がイメージしやすいです。

参考） [ファイルディスクリプタとは？](https://wa3.i-3-i.info/word14383.html)

つまり、 `ファイルを識別する仕組み` です。
また、ファイルを識別する仕組みにはOS毎に上限があります。

## OSのファイルを扱える数とは？
OSごとに、いくつファイルを扱えるか限界が決まっています。こればっかりは確認しないとわからないです。

### 確認

Nginxが動いているサーバーに入って確認してください。

```perl
$ cat /proc/sys/fs/file-max
524288
```

`524288` これがOSで扱える最大ファイル数になります。

### 何故確認する必要がある？
OSで扱えるファイル数以上のファイルディスクリプタをNginxで使う設定をしてしまうと、NginxでOSで扱えない量のファイル数を見に行ってしまうのでエラーになる。
超えない値に調整する必要があり、そのために現状のOSで扱える最大ファイル数を確認する必要があります。

参考）http://please-sleep.cou929.nu/nginx-too-many-open-files-error.html

## worker_connectionsのプロセスがいくつ動いているか？

### 確認
nginx.conf に設定できる箇所があるので、ここに設定していればその数がworkerプロセス数です。

```perl
# これだとworkerプロセスは6個
worker_processes 6;

# これだとCPUコア数で自動でプロセス数をNginxがいい感じにしてくれているので調べる必要あり
worker_processes auto;
```

自動でいい感じのプロセスを立ててくれる auto にしている人が多いと思うので、確認しましょう。 psやtopなど。

```perl
$ top
    8     1 nginx    S    14680   1%   0   0% nginx: worker process
    7     1 nginx    S    14680   1%   4   0% nginx: worker process
   10     1 nginx    S    14680   1%   0   0% nginx: worker process
    9     1 nginx    S    14680   1%   0   0% nginx: worker process
   11     1 nginx    S    14680   1%   1   0% nginx: worker process
   12     1 nginx    S    14680   1%   4   0% nginx: worker process
    1     0 root     S    13800   1%   2   0% nginx: master process nginx -c /etc/nginx/nginx.conf -g daemon off;
```

`workerプロセスは6` です。

### 何故確認する必要がある？
worker_rlimit_nofileは、1workerプロセスにおけるファイルディスクリプタの上限値です。
なので、workerプロセスが何個動いているかを確認しないと適切な値が設定できないためです。

## 改めて、どの値にすべき？
上記の話を踏まえて、worker_rlimit_nofileは以下のように決定します。

`workerプロセス数 * 設定する予定のworker_rlimit_nofileの値 < OS全体で扱えるファイル数以下の値`

逆算したらいい値が出せます。

`OS全体で扱えるファイル数 / workerプロセス数 = 設定すべきworker_rlimit_nofileの値`

今回でいうと、 524288 / 6 = 87381 で、 `87381` になります。

ただ、上限ギリギリは良くないので 5% くらい減らして上げましょう。（5％という値は自分の好みですが）

`87381 * 0.95 = 83012` なので、キリよく `83000` にしました。

# worker_connectionsの値は何にすべき？
## そもそもworker_connectionsとは？
Nginxのworker_connectionsの1workerプロセスにおける、 `ファイルディスクリプタ` の上限値

引用） [Nginx - worker_connecions](http://nginx.org/en/docs/ngx_core_module.html#worker_connections)
> Sets the maximum number of simultaneous connections that can be opened by a worker process.
> 和訳） workerプロセスによって開くことができる同時接続の最大数を設定する

## 適切な値を考える
で、いくつに設定すればいいかと言う事になりますが、以下を守っていれば基本的に問題ありません。

`worker_connections * 2 < worker_rlimit_nofile`

### 何故 worker_connections の2倍の値が worker_rlimit_nofile 以下であればOK？
１接続で１ファイルディスクリプタしか使わないんじゃないの？って思うかもしれませんが、1接続で2ファイルディスクリプタを使います。
通常、1つの接続につき以下のファイルディスクリプタを消費します。

1. エフェメラルポート用のソケットファイル
2. 実際に返答するコンテンツファイル

また、これは通常のケースで設定やバックエンドシステムによってはもっと増える可能性があります。なので、基本的には設定したファイルディスクリプタの半分の値です。
一応余裕を持って3~4倍がいいと各所で言われていますね。その通りだと思います。

## 改めて、どの値にすべき？
今回、自分が発生したエラーは以下です。

```perl
2048 worker_connections are not enough
```

なので、これ以上の値にする必要があります。 今回はworker_connectionsを2倍に増やして `4096` とかにしてみましょうか。

また、 4096 の4倍はファイルディスクリプタを消費する可能性があるので、 `16384` のworker_rlimit_nofileの値があれば大丈夫です。

先ほど設定したworker_rlimit_nofileは `83000` でした！ なので、余裕で大丈夫です。

最終的に、僕の nginx.conf は以下の通りになりました。

```perl
worker_rlimit_nofile 83000; # worker_connections の2~4倍以上の値を設定。OS全体で扱うことができるファイル数を超えないように注意
 
events {
  worker_connections 4096; # 1つのworkerプロセスが開ける最大コネクション数
  ...
}

...
```

# まとめ

以下の計算式を守っていれば大丈夫！
### worker_rlimit_nofileの値の決め方
`worker_rlimit_nofile * workerプロセス数 < OS全体で扱えるファイル数`

### worker_connectionsの値の決め方
`worker_connections * 4 < worker_rlimit_nofile`

わかっているとは思いますが、この設定によってNginxが受け入れるコネクション数が増えるので、後ろに続くバックエンドの処理への負荷は上昇します。

そちらのCPUも考慮して上げてくださいね！ それではまた。