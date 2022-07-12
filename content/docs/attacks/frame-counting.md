+++
title = "Frame Counting"
description = ""
date = "2020-10-01"
category = [
    "Attack",
]
abuse = [
    "iframes",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
    "COOP",
    "Framing Protections",
]
menu = "main"
weight = 2
+++

ウィンドウの参照により、クロスオリジンのページが他のページのいくつかの属性にアクセスできます。
これらの参照は、`iframe`と`window.open`を使用または許可するときに利用可能となります。
この参照は、同一生成元ポリシーを尊重し続けるため、ウィンドウに関する（限定的な）情報を提供します。

アクセス可能な属性の1つは、ウィンドウ内のフレーム数を提供する`window.length`です。
この属性は、ページに関する貴重な情報を攻撃者に提供します。

Webサイトでは一般的にフレーム（または`iframes`）を使用しますが、この選択は必ずしもセキュリティ上の問題を意味するわけではありません。
しかし、Webサイトがユーザの情報に応じてページのフレーム数を変更する場合があります。
例えば、`GET`パラメータと利用者のデータに応じてレイアウトを変えるようなページで起こりえます。
攻撃者は、異なるページで `window.length` の値を測定することにより、被害者に関する情報を推測することが可能かもしれません。

## コード

以下のコードは、クロスサイトのページにおけるフレーム数に関する情報にアクセスする方法を示しています。

```javascript
// ウィンドウへの参照を取得する
var win = window.open('https://example.org');

// ページが読み込まれるのを待つ
setTimeout(() => {
  // 読み込まれたiframeの数を読み取る
  console.log("%d iframes detected", win.length);
}, 2000);
```

## 攻撃の対策

場合によっては、異なるアプリケーション状態が同じ数のフレームを持つことで、攻撃者がそれらを区別できないようにすることができます。
ただし、ページの読み込み中にフレーム数を連続的に記録することで、攻撃者に情報をリークする可能性があるパターンを示す可能性もあります。

```javascript
// ウィンドウへの参照を取得する
var win = window.open("https://example.org");
var pattern = [];

// ループ内で、60ms間隔でiframeの数を登録する
var recorder = setInterval(() => {
  pattern.push(win.length)
}, 60);

// 6秒後にループを解除する
setTimeout(() => {
   clearInterval(recorder);
   console.log("The pattern is: %s", pattern.join(', '));
}, 6 * 1000);
```

## 事例

フレームカウント攻撃の例としては、以下のようなものがあります。

- あるWebサイトでは、ユーザが検索エンジンでユーザ情報を検索することができます。もしユーザの検索に対する結果があるかどうかによって、ページ構造の`iframe`の数が異なる場合に、攻撃者は[XS-Search]({{< ref "xs-search.md" >}})の技術を利用して、秘密をリークさせることができるでしょう。
- あるWebサイトでは、性別やその他の個人情報に基づいてユーザプロファイルページの構造が異なります。攻撃者は、ページを開いてフレームをカウントすることで、情報を簡単にリークさせることができるでしょう。

## 対策

| Attack Alternative | [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |    [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})    |
| :----------------: | :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :-------------------------------------------------------------------------: |
|      iframes       |                                         ✔️                                          |                          ❌                          |                                 ✔️                                 |  [FIP]({{< ref "/docs/defenses/isolation-policies/framing-isolation" >}})   |
|      windows       |                                         ❌                                          |                          ✔️                          |                                 ❌                                 | [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |

## 実例

Facebookに報告された脆弱性では、この手法を利用することで、投稿に掲載された特定の内容、友人の宗教情報、写真の位置情報などのユーザに関する情報をリークするというものでした[^1]。

## 参考文献

[^1]: Patched Facebook Vulnerability Could Have Exposed Private Information About You and Your Friends. [link](https://www.imperva.com/blog/facebook-privacy-bug/)
