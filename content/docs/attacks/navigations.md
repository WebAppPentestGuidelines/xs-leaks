+++
title = "Navigations"
description = ""
date = "2020-10-01"
category = [
    "Attack",
]
abuse = [
    "Downloads",
    "History",
    "CSP Violations",
    "Redirects",
    "window.open",
    "window.stop",
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

クロスサイトのページで画面遷移がトリガーされたか（または、そうでないか）を検出することは攻撃者にとって有用です。例えば、ウェブサイトはユーザの状態に依存({{< ref "#case-scenarios" >}})して、あるエンドポイントで画面遷移をトリガーする可能性があります。

どのような画面遷移が発生したかを検出することで攻撃者以下の様なことが可能になります。

- `iframe`を使用して`onload`イベントがトリガーされた回数を数える。
- ウィンドウへの参照を通じてアクセス可能な`history.length`の値をチェックする。この値は被害者の`history.pushState`や通常の画面遷移によって変更された履歴の数を提供しています。攻撃者は`history.length`の値を取得するためにウィンドウへの参照のlocationをターゲットのウェブサイトに変更し、そしてSame-Originに戻すことによって最後に値を読み取ります。

## ダウンロードトリガー

エンドポイントが[`Content-Disposition: attachment`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition) ヘッダを設定すると、ブラウザにレスポンスをナビゲートさせるのではなくファイルとしてダウンロードすることを指示します。この挙動が発生したかを検出すると、結果が被害者の状態に依存する場合に攻撃者に機密情報をリークできる可能性があります。

### ダウンロードバー

Chromeベースのブラウザではファイルをダウンロードする際に、ブラウザのウィンドウ下部にダウンロードの進捗を示すバーがウィンドウと一体化して表示されます。攻撃者ウィンドウの高さを監視することでダウンロードバーが開いているかどうかを検出することができます。


```javascript
// ウィンドウの現在の高さを読み取る
var screenHeight = window.innerHeight;
// ダウンロードのトリガーとなるページを読み込む
window.open('https://example.org');
// タブの読み込みを待つ
setTimeout(() => {
    // ダウンロードバーが表示された場合、すべてのタブの高さが小さくなります
    if (window.innerHeight < screenHeight) {
      console.log('Download bar detected');
    } else {
      console.log('Download bar not detected');
    }
}, 2000);
```

{{< hint important >}}
この攻撃は、自動ダウンロード機能が有効になっているChromeベースのブラウザでのみ有効です。加えてこの攻撃はユーザがダウンロードばーばを能動的に閉じないと再検出できないため、繰り返し行うことはできません。
{{< /hint >}}

### iframeを利用したダウンロード遷移

[`Content-Disposition: attachment`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition) ヘッダをテストするもう一つの方法は遷移が発生したかどうかをチェックすることです。仮にページの読み込みによってダウンロードが発生した場合、遷移は発生せずウィンドウは同一オリジン内に留まります。

以下のコードは、そのような遷移が発生したかを検出しダウンロードが試行されたかを検出します。

```javascript
// ダウンロード試行時のテスト先URLを設定する
var url = 'https://example.org/';
// onloadイベントを計測するための外側のiframeを作成する
var iframe = document.createElement('iframe');
document.body.appendChild(iframe);
// ダウンロードの試行をテストするために、内側のiframeを作成
iframe.srcdoc = `<iframe src="${url}" ></iframe>`;
iframe.onload = () => {
      try {
          // ナビゲーションが発生した場合、iframeはクロスオリジンになるため、"inner.origin "にアクセスすると例外が発生します。
          iframe.contentWindow.frames[0].origin;
          console.log('Download attempt detected');
      } catch(e) {
          console.log('No download attempt detected');
      }
}
```

{{< hint info >}}
ダウンロードを試行することにより`iframe`内で画面遷移が発生しない場合、`iframe`は`onload`イベントを直接トリガーしません。そのため、上記の例では代わりに外側に`iframe`を使用し、その中の`iframe`を含むサブリソースの読み込みが完了した際にトリガーされる`onload`イベントを待ち受けます。
{{< /hint >}}

{{< hint important >}}
この攻撃はどのような[Framing Protections]({{< ref "xfo" >}}にかかわらず動作します。 `Content-Disposition: attachment`が指定されると`X-Frame-Options`と`Content-Security-Policy`ヘッダは無視されるからです。
{{< /hint >}}

### ダウンロード遷移（iframeなし）

前項で紹介した手法は`window`オブジェクトを利用しても効果的にテストすることができます。

```javascript
// 送信先URLをセット
var url = 'https://example.org';
// windowへの参照を取得
var win = window.open(url);

// windowが読み込まれるまで待機
setTimeout(() => {
      try {
          //  ナビゲーションが発生した場合、iframeはクロスオリジンになるため、"win.origin" にアクセスすると例外が発生する
          win.origin;
          parent.console.log('Download attempt detected');
      } catch(e) {
          parent.console.log('No download attempt detected');
      }
}, 2000);
```

## サーバサイドリダイレクト

### 拡張

サーバサイドリダイレクトは、宛先URLのサイズと攻撃者によって制御されている入力値（クエリ文字列またはパスのいずれか）が増加する場合、クロスオリジンのページから検出することができます。以下の手法は大きなクエリ文字列やパスを生成する子男でほとんどのWEBサーバでエラーを誘発することが可能であるという事実に依存しています。リダイレクトはURLのサイズを増加させるため、サーバが処理可能なURLの最大長より正確に1文字減らすることで検出することができます。これによりサイズが大きくなった場合、サーバはクロスオリジンのページから検出可能なエラーを応答します。（例えばエラーイベント経由）

{{< hint example >}}
攻撃の例はこちらで確認することができます。 [here](https://xsleaks.github.io/xsleaks/examples/redirect/).
{{< /hint >}}

## クロスオリジンリダイレクト

### CSP 侵害

[Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) (CSP) はクロスサイトスクリプティングやデータインジェクションに対する綿密な防御機構です。CSPが侵害されると`SecurityPolicyViolationEvent` がトリガーされます。攻撃者は[`connect-src` ディレクティブ](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/connect-src) を利用してCSPを設定することができます。このディレクティブは`fetch` がCSPディレクティブで設定されていないURLを追うたびに`Violation`イベントがトリガーされます。これによって攻撃者は他のオリジンへのリダイレクトが発生したかを検出することができます。

以下の例では、fetch APIで設定（6行目）したウェブサイトが`https://example.com`以外のウェブサイトにリダイレクトされると`SecurityPolicyViolationEvent` がトリガーされます。

{{< highlight html "linenos=table,linenostart=1" >}}
<!-- Set the Content-Security-Policy to only allow example.org -->
<meta http-equiv="Content-Security-Policy"
      content="connect-src https://example.org">
<script>
// Listen for a CSP violation event
document.addEventListener('securitypolicyviolation', () => {
  console.log("Detected a redirect to somewhere other than example.org");
});
// Try to fetch example.org. If it redirects to another cross-site website
// it will trigger a CSP violation event
fetch('https://example.org/might_redirect', {
  mode: 'no-cors',
  credentials: 'include'
});
</script>
{{< / highlight >}}

攻撃対象のリダイレクトがクロスサイトであることと、`SameSite=Lax`と設定されたクッキーの存在が条件である場合、`fetch`はトップレベルの遷移としてカウントされないため上記の手法は動作しません。このような場合、攻撃者は別のCSPディレクティブである[`form-action`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/form-action)と、`GET`を使ってHTMLフォームを送信するとトップレベル遷移としてカウントされるという事実を利用することができます。

以下の例では、フォームのアクション（3行目）が`https://example.org`以外にリダイレクトされると`SecurityPolicyViolationEvent`がトリガーされます。

{{< highlight html "linenos=table,linenostart=1" >}}
<!-- Content-Security-Policyをexample.orgのみ許可するように設定する -->
<meta http-equiv="Content-Security-Policy"
      content="form-action https://example.org">
<form action="https://example.org/might_redirect"></form>
<script>
// CSP違反イベントをリッスンする
document.addEventListener('securitypolicyviolation', () => {
  console.log("Detected a redirect to somewhere other than example.org");
});
// フォームからexample.orgを取得してみてください。別のクロスサイトなウェブサイトにリダイレクトされると、CSP違反のイベントが発生する
document.forms[0].submit();
</script>
{{< / highlight >}}

FireFoxではChromeベースのブラウザとは異なり`form-action`がform送信後にリダイレクトをブロックしないため、この手法は実行できないことに注意してください。

## 攻撃シナリオ

とあるオンラインバンクでは富裕層ユーザが口座の残高を確認した際に、ウェブサイト上の予約ページへナビゲーションをトリガーすることにより魅力的な株の投資機会へリダイレクトすることになっています。このように特定のユーザグループに対してのみ行われている場合、攻撃者に顧客ステータスがリークされます。

## 分割されたHTTPキャッシュの回避

もし、サイト`example.com` が `*.example.com/resource` からのリソースを含む場合、sのリソースはトップレベルナビゲーションを通して直接リクエストされた場合と同じキャッシュキーを持つことになります。これは、キャッシュキーがトップレベルの *eTLD+1* と フレームの *eTLD+1* で構成されるからです。[^cache-bypass]

リソースがキャッシュされているかどうかは `window.stop()` が実行される前にオリジンが変更されたかどうかをチェックすることによって検出することができるのは、ウィンドウは`window.stop()` で異なるオリジンへのナビゲーションを防ぐことができ、デバイス上のキャッシュはネットワークより高速であるためです。

```javascript
async function ifCached_window(url) {
  return new Promise(resolve => {
    checker.location = url;

    // キャッシュのみ
    setTimeout(() => {
      checker.stop();
    }, 20);

    // 結果を取得
    setTimeout(() => {
      try {
        let origin = checker.origin;
        // タイムアウト前にOriginが変更されていない
        resolve(false);
      } catch {
        // Originが変更された
        resolve(true);
        checker.location = "about:blank";
      }
    }, 50);
  });
}
```
windowを作成（チェック成功後に戻ることが可能になる）
```javascript
let checker = window.open("about:blank");
```
使い方
```javascript
await ifCached_window("https://example.org");
```
{{< hint info >}}
分割されたHTTPキャッシュを回避するにはヘッダ `Vary.Sec-Fetch-Site` を使って防ぐことができます。`Sec-Fetch-Site` はキャッシュを開始者によって分割するため、[Cache Protections]({{< ref "/docs/defenses/design-protections/cache-protections.md" >}}) を参照してください。この攻撃は同じサイトのリソースにのみ適用されるため、`Sec-Fetch-Site`ヘッダーはウェブサイトの `same-site` や `same-origin` に対して、攻撃者にとっては `cross-site` になるため、うまくいきます。
{{< /hint >}}

## 対策

|       Attack Alternative        | [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |                                          [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})                                          |
| :-----------------------------: | :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------: |
|   *history.length* (iframes)    |                                         ✔️                                          |                          ❌                          |                                 ✔️                                 |                                        [FIP]({{< ref "/docs/defenses/isolation-policies/framing-isolation" >}})                                         |
|   *history.length* (windows)    |                                         ❌                                          |                          ✔️                          |                                 ❌                                 |                                       [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}})                                       |
| *onload* event inside an iframe |                                         ✔️                                          |                          ❌                          |                                 ✔️                                 |                                        [FIP]({{< ref "/docs/defenses/isolation-policies/framing-isolation" >}})                                         |
|          Download bar           |                                         ✔️                                          |                          ❌                          |                  ❌{{< katex>}}^{1}{{< /katex >}}                  |                                       [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}})                                       |
|  Download Navigation (iframes)  |                                         ✔️                                          |                          ❌                          |                  ❌{{< katex>}}^{1}{{< /katex >}}                  |                                        [FIP]({{< ref "/docs/defenses/isolation-policies/framing-isolation" >}})                                         |
|  Download Navigation (windows)  |                                         ❌                                          |           ❌{{< katex>}}^{1}{{< /katex >}}           |                                 ❌                                 |                                       [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}})                                       |
|            Inflation            |                                         ✔️                                          |                          ❌                          |                                 ❌                                 |                                        [RIP]({{< ref "/docs/defenses/isolation-policies/resource-isolation" >}})                                        |
|         CSP Violations          |            ❌{{< katex>}}^{2}{{< /katex >}}                                        |                          ❌                          |                                 ❌                                 | [RIP]({{< ref "/docs/defenses/isolation-policies/resource-isolation" >}}) 🔗 [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |

🔗 – Defense mechanisms must be combined to be effective against different scenarios.

____
1. [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) と [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) のどちらも、 `Content-Disposition` ヘッダが存在すると他のヘッダが無視されてしまい、リダイレクトリークを緩和することはできません。
2. LaxモードのSameSite Cookieは、ウェブサイトのiframingから保護することができますが、Strictモードとは対照的に、ウィンドウへの参照やサーバー側のリダイレクトを含むリークには役立ちません。

## Real-World Examples

Twitterに報告された脆弱性は、この手法を利用して[XS-Search]({{< ref "../attacks/xs-search.md" >}})による非公開ツイートの内容を漏洩させるというものでした。この攻撃は、ユーザーのクエリ[^1]に対する結果がある場合にのみ、ページがナビゲーションを開始するため、可能でした。

## References

[^1]: Protected tweets exposure through the url, [link](https://hackerone.com/reports/491473)
[^2]: Disclose domain of redirect destination taking advantage of CSP, [link](https://bugs.chromium.org/p/chromium/issues/detail?id=313737)
[^3]: Using Content-Security-Policy for Evil, [link](http://homakov.blogspot.com/2014/01/using-content-security-policy-for-evil.html)
[^cache-bypass]: [github.com/xsleaks/wiki/pull/106](https://github.com/xsleaks/wiki/pull/106)
