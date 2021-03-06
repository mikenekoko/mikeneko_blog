---
title: windowsとmacのキーボード配置がバラバラになっていてタイプミスが多発するのでキー配列をいじった話
date: 2019-04-30 12:07:57
tags:
- Mac
- Windows
- キーボード
- 配置変更
---

会社のPCがMACになりました。家ではLenovoのWindowsPCを使っています。
それ自体は別にいいんですが（どちらのOSも利点はあるため）キーボード配列が一部違うのが本当にイライラします！ｗ
脳内で、「今はWindowsだからCtrlキーはここだな…」とか、「MACは半角／全角ボタンが無いから…」みたいに考えなきゃいけないのが本当に辛いです。

ってことでキー配列をある程度合わせてしまいましょう！

# そもそものキーボードの違い
### Windows
![key.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/178351/4019b368-a1f6-6a08-ac77-3601a079f6d1.jpeg)


### MAC
![2016-11-02-12.28.49.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/178351/4463f9d8-ae55-2cbe-c99d-678424aadaaa.png)

> ちなみに、この記事はMACのキーボードはJIS配列前提で書いています。日本語変換とか明らかにこっちの方がやりやすいと思うんですよね。好みですが。

赤い線の箇所が大幅に間違える箇所です！
表にすると以下です。

### Windows

| キー名 | 場所 | 役割 |
|:--|:--|:--|
| CapsLock | 左中央 | 英語/かな切り替え<br>Shitと同時押しで大文字英語に固定 |
| 無変換 | スペース左 | 不明。使った事ない。 |
| 変換 | スペース右 | 不明。使った事ない。 |
| カタカナ<br>ひらがな<br>ローマ字 | スペース2右 | Shift+このキーでカタカナ入力<br>Alt＋このキーで日本語入力 |

### MAC
| キー名 | 場所 | 役割 |
|:--|:--|:--|
| control | 左中央 | Ctrlキーと同様 |
| 英数 | スペース左 | 英語切り替え |
| かな | スペース右 | かな切り替え |

はい、なぜ押し間違えるかというと表をみたらわかると思うのですが、 *MACだと重要なキーの配置場所にWindowsだとゴミなキーが配置されている* という現象が起きています。
別に今までWindows一筋だったときはこんなキー使わなかったからよかったんですよ。今も使っていませんし。
ですが、MACだと有効なキーです。親指でポチポチと日本語と英語を切り替えられCtrlキーがめちゃくちゃ便利な左側に配置されている。これは使いますよね。そうしていると、家でWindowsを使った時に間違えて触ってしまうわけです。

どちらかに合わせる必要があるのですが、明らかに *MACのキーボード配列の方が優秀* なので、こちらに配置を合わしてしまいましょう！

# Windowsのキー配列を変更する
物理的にキーボードを改造するわけではないですよ？ｗ 自分のPCはノートPCなので現実的ではないですしね。
## CapsLockをCtrlキーにする
これはレジストリを直接変更するか、プラグインを入れる必要があります。レジストリ直接変更は危険なので、プラグインを入れる方式でやる方を絶対おすすめします。そっちも危ないんじゃないの？っておもうかもしれませんが、こっちはMicrosoftが直に出しているものになります！

リンクは[こちら](https://docs.microsoft.com/ja-jp/previous-versions/bb897578)からなのですが、なんと配布が終わっていました…

![キャプチャ.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/178351/625b41ed-32d3-89d0-bde4-eb5e941c9db8.png)

ですが、改良版がでていたようです。[こちら](https://docs.microsoft.com/en-us/sysinternals/downloads/ctrl2cap)からダウンロードしましょう！

ダウンロードしたらインストールして再起動。CapsLockがCtrlになっていれば成功です！

## 無変換/変換/カタカナひらがなローマ字キーを変更する
MicrosoftIMEの設定を変更します。GoogleIME等を使っている人は、適宜読み替えてください。
IMEを右クリックして、プロパティへと進みます。

![windows10-ime-03-480x562.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/178351/9bf3f6c8-863c-3142-7955-6c38083c461d.png)

詳細設定をクリック
![windows10-ime-03a-480x517.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/178351/9bce0eea-1576-0343-4c90-3365b9f0d9fb.png)

編集操作から、変更へ進みます。
![windows10-ime-04-480x521.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/178351/c73b16e4-c6ab-fada-a5a9-bd8d350cbc71.png)

ここの設定を以下のように変更します！
![キャプチャ1.PNG](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/178351/b0d320f3-50b9-f034-079b-462bea729e63.png)

* IME-オフ : 英字入力
* IME-オン : かな入力

となっています。これでMACと同じキー配列にすることができました！

# 変更してみて
やっぱりまだ心のどこかで前のWindowsの操作がしみついているのか、たまにCapsLockを英語とかな入力の切り替えで押してしまったりする自分がいます…ｗ
ですが、これからこの配置に統一することでMACとWindowsの差異を無くすことができ、かつ効率的なタイピングができるはずです！頑張って慣れようと思います。

同じくキーボード配置に悩まれている方は是非！とても快適です。
それではまた。