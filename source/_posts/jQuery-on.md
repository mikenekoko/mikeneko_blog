---
title: jQueryの .on ではオブジェクトを渡すことができない！
date: 2019-04-07 03:42:45
tags:
- jQuery
---

スマホでスクロールを禁止させる処理を追加しました。

```javascript
$(document).on('touchmove', function(e) {e.preventDefault();});
```

ところが、GoogleChromeだけ動作しない。コンソールに以下の警告が表示されていました。

```javascript
[Intervention] Unable to preventDefault inside passive event listener due to target being treated as passive. 
See https://www.chromestatus.com/features/5093566007214080
preventDefault @ jquery-1.11.2.min.js:3 (anonymous)
```

中に表示されているURLをクリックして、中身を確認します。
https://www.chromestatus.com/features/5093566007214080

```java
AddEventListenerOptions defaults passive to false. 
With this change touchstart and touchmove listeners added to the document will default to passive:true 
(so that calls to preventDefault will be ignored)..

If the value is explicitly provided in the AddEventListenerOptions it will continue having the value specified by the page.

This is behind a flag starting in Chrome 54, and enabled by default in Chrome 56. 
See https://developers.google.com/web/updates/2017/01/scrolling-intervention
```

要するに、Chrome56以降はデフォルトで `passive：true` になっているからスクロールできないよ！って言っています。
これはfalseである必要があるので、 `passive: false` を渡してあげましょう！

# passive: false を .on に渡すとエラーになる
passive: false を渡す記述に変更。

```javascript
$(document).on('touchmove', function(e) {e.preventDefault();}, {passive: false});
```

とすると、以下のエラー。

```javascript
TypeError: ((m.event.special[e.origType] || {}).handle || e.handler).apply is not a function
```

？？？エラーの意味がよくわかりませんね。。
jQueryの公式リファレンス（ http://api.jquery.com/on/ ）を見ると、オブジェクトとして `passive: false` を渡すことは不可能なようです。以下のように明記されています。

```java
.on(events[,selector][,data],handler)
```

今回自分がやろうとしてるのは以下の形式になるので、不可能ですね。

```javascript
.on(events[,selector][,data],handler,第3引数でオブジェクトを渡す)
```

# 仕方ないので素のJSで渡す
素のJSのAddEventListenerのリファレンス（https://developer.mozilla.org/ja/docs/Web/API/EventTarget/addEventListener ）では、第三引数としてオブジェクトを渡すことができるようです！

```javascript
target.addEventListener(type, listener[, options]);
```

記述方法をjQyeruから素のJSに変更します。

```javascript
document.addEventListener('touchmove', function(e) {e.preventDefault();}, {passive: false});
```

エラー無し。スクロール制御ができました！
こちらの記述方法であれば第三引数にオブジェクトを渡すことができます。

# 最後に
この方法はエラーを回避しただけで、ここだけ素のJSで書いてしまってコードレビュー時に色々と問題になりましたｗ
後から見た人はなんで素のJSで書かれているかわかりませんからね。

本当はjQueryの.onで解決したかったので、もしこうすれば解決できるよ！ってのがわかる方いましたらアドバイス頂けますと非常に助かります。