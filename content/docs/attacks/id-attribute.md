+++
title = "ID属性"
description = ""
date = "2022-05-10"
category = [
    "Attack",
]
abuse = [
    "onblur",
    "focus",
    "iframes",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
    "Framing Protections",
    "Document Policies"
]
menu = "main"
weight = 3
+++

`id`属性はHTML要素を識別するために広く利用されています。残念ながらクロスオリジンのウェブサイトで`focus`イベントと`URL`フラグメントを利用することによって、ページのどこに`id`属性が与えられているかを決定することができます。例えば、`https://example.com/foo#bar`が読み込まれると、ブラウザは`id="bar"`が与えられた要素までスクロールしようとします。これはクロスオリジンのサイトで定義したiframeで`https://example.com/foo#bar`を読み込むことによって検出することができます。もし、 `id="bar"` を持つ要素がある場合は、 `focus` イベントが発生します。 `focus` イベントもまた同じ目的で使用することができます[^1]。

いくつかのウェブアプリケーションは`focusable`要素に`id`属性を設定しており、ユーザ情報の開示につながります。これらの`id`属性には、ユーザに直接関連する機密情報やユーザの状態（アカウントの状態）に関連する情報を含めることができます。

## コードスニペット

以下のコードは別のサイトからID属性を検出する例を示しています：
```javascript
// Listen to onblur event
onblur = () => {
  alert('Focus was lost, so there is a focusable element with the specified ID');
}
var ifr = document.createElement('iframe');
// If a page has a focusable element with id="x" it will gain focus
// E.g. <input id="x" value="test" />
ifr.src = 'https://example.org/#x';
document.body.appendChild(ifr);
```

{{< hint info >}}
上記の手法はFireFoxではうまく動作しない可能性があります。
{{< /hint >}}

## 攻撃シナリオ

`id`属性を利用した攻撃は以下のような物があります。
- とある銀行がモバイルデバイスのセッションを認証するために短い数字のワンタイムPin(OTP)を生成することを許可しています。この銀行のページはクライアントにPINコードを表示するための`button`要素の`id`にOTPコードそのものを使用していました。この挙動を悪用し、すべてのオプションをブルートフォースすることでOTPコードを窃取し、ユーザアカウントを侵害することができます。
- とあるウェブアプリケーションが、プレミアムアカウントのステータスを持つユーザや、特定の性別のユーザである場合、あらかじめ定義された`id`と`HTML`要素の組み合わせを利用します。攻撃者は、被害者のページに特定の`id`があるかを検出し、被害者のアカウント情報を漏えいさせることができます。

## Defense

| [Document Policies]({{< ref "/docs/defenses/opt-in/document-policies.md" >}}) | [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |                                          [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})                                          |
| :--------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------: |
|                                         ✔️                                          |                                         ✔️                                          |                          ✔️                          |                                 ❌                                 | [FIP]({{< ref "/docs/defenses/isolation-policies/framing-isolation" >}}) |


## References

[^1]: Leaking IDs using focus, [link](https://portswigger.net/research/xs-leak-leaking-ids-using-focus)
