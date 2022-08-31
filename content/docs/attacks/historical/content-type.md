+++
title = "Content-Type"
description = ""
date = "2020-10-01"
category = "Historical"
abuse = [
    "typeMustMatch",
    "iframes",
    "Content-Type",
    "Status Code",
]
defenses = [
    "Deprecation"
]
menu = "main"
+++

リクエストのContent-Typeをリークすることで、攻撃者は2つのリクエストを区別できるようになります。

## typeMustMatch

[`typeMustMatch`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLObjectElement/typeMustMatch)は`object`要素の`typeMustMatch`属性を反映したブール値です。これは、オブジェクトを読み込む際に、リソースの`Content-Type`がオブジェクトで提供されるものと同じかどうかを検証することで、特定の MIME タイプを強制しなければならないことを保証します。残念ながら、この強制は攻撃者がウェブサイトから返される`Content-Type`とステータスコードを漏らすことも可能にします [^1]。

### 根本的な原因

以下のスニペットを考えると、`https://target/api` で返された `Content-Type` が `type` のものと一致しない場合や、サーバが `200` 以外のステータスを返した場合には、 `not_loaded` がレンダリングされるでしょう。

```html
<object type="application/json"
        data="https://example.org"
        typemustmatch>
not_loaded </object>
```

#### 問題

攻撃者は、[すべての条件]({{< ref "#root-cause" >}})が満たされたときに起こるオブジェクトのレンダリングを検出することによって、ウェブサイトの`Content-Type`とステータスコードをリークできます。攻撃者は、(ステータスコード`200`で)オブジェクトがレンダリングされるとき、0ではないであろう`clientHeight`と`clientWidth`の値をチェックできます。`typeMustMatch`はリソースを読み込む際に、サーバーがステータス`200`を返すことを要求するので、[Error Events]({{< ref "../error-events.md" >}}) XS-Leaksと同様にエラーページを検出することが可能でしょう。

以下の例では、`iframe`内にオブジェクトを埋め込み、`iframe`が`onload`イベントをトリガーしたときに`clientHeight`と`clientWidth`の値をチェックすることでこの動作を検出する方法を示しています。


```javascript
// 配送先のWebサイトのURLを設定する
var url = 'https://example.org';
// チェックしたいコンテンツタイプ
var mime = 'application/json';
var ifr = document.createElement('iframe');
// オブジェクトがonloadイベントを発生させないので、iframe内にオブジェクトをロードする。
ifr.srcdoc = `
  <object id="obj" type="${mime}" data="${url}" typemustmatch>
    error
  </object>`;
document.body.appendChild(ifr);

// iframeが読み込まれたら、オブジェクトの高さを読み取ります。
// もしそれが一行のテキストの高さであれば、リソースのContent Typeは`application/json`ではなかったということです。
// もしそうでなければ、それは`application/json`だったということです。
ifr.onload = () => {
    console.log(ifr.contentWindow.obj.clientHeight)
};
```

### 修正

Firefox は `typeMustMatch` 属性をサポートする唯一のブラウザでした [^2] が、他のブラウザがサポートを提供しなかったため、バージョン 68で削除され、HTML Living Standard からも削除されました。

## 参考文献

[^1]: Cross-Site Content and Status Types Leakage, [link](https://medium.com/bugbountywriteup/cross-site-content-and-status-types-leakage-ef2dab0a492)
[^2]: Remove support for typemustmatch, [link](https://bugzilla.mozilla.org/show_bug.cgi?id=1548773)
