+++
title = "Error Events"
description = ""
date = "2020-10-01"
category = [
    "Attack",
]
abuse = [
    "Error Events",
    "Status Code",
    "nosniff",
    "Content-Type",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
]
menu = "main"
weight = 2
+++


Webページがサーバにリクエストを発行すると（例：フェッチ、HTMLタグ）、サーバはこのリクエストを受信して処理します。
受信した際にサーバは与えられた条件に基づいて、リクエストが成功（例：200）すべきか失敗（例：404）すべきかを決定します。
レスポンスがエラーステータスを持つ場合、ページを処理するためにブラウザによって[error event](https://developer.mozilla.org/en-US/docs/Web/API/Element/error_event)が発行されます。
これらのエラーは、例えば`HTML`コンテンツを画像として埋め込もうとした時のようなパーサーの処理が失敗するケースにも対応しています。

たとえば、攻撃者は認証されたユーザだけが利用できるリソースにユーザがアクセスできるかどうかを確認することで、ユーザがサービスにログインしているかどうかを検出できます[^3]。
このXS-Leakの影響はアプリケーションによって異なりますが、ユーザを非匿名化する高度な攻撃につながる可能性があります[^1]。

エラーイベントは様々な HTMLタグからスローされる可能性があり、ブラウザによって動作が異なるものもあります [^4]。
例えば、読み込まれたリソースやHTMLタグ、特定のヘッダ（例えば `nosniff` や `Content-Type` ）の存在、あるいはブラウザのデフォルトの保護機能の適用などによって動作が変わることがあります。

エラーイベントで情報をリークさせる原理は、さまざまなXS-Leakに抽象化して適用できます。 
たとえば、[Cache Probing]({{< ref "cache-probing.md" >}})の1つの手法では、特定の画像がブラウザによってキャッシュされたかどうかの検出にエラーイベントを使用しています。

## コード
以下のコードは、`<script>` タグを使用してエラーイベントを検出する方法を示しています。

```javascript
function probeError(url) {
  let script = document.createElement('script');
  script.src = url;
  script.onload = () => console.log('Onload event triggered');
  script.onerror = () => console.log('Error event triggered');
  document.head.appendChild(script);
}
// google.com/404 が HTTP 404 を返すため、スクリプトはエラーイベントを発生する
probeError('https://google.com/404');

// google.com は HTTP 200 を返すので、スクリプトは onloadイベントをトリガーする
probeError('https://google.com/');
```

## 対策

このXS-Leakの緩和策は、アプリケーションが特定のリソースをどのように処理するかによって異なる可能性があります。
一般的なアプローチとしては、可能な限り一貫した挙動を採用することです。
特定のシナリオでは、アプリケーションは[Subresource Protections]({{< ref "/docs/defenses/design-protections/subresource-protections.md" >}})を使用して、攻撃者がURLを予測して攻撃を進行させることを防ぐことができます。
最後に、一般的なWebプラットフォームのセキュリティ機能を導入することで、アプリケーションの実装に大きな変更を加えることなく、このXS-Leakをより大幅に緩和することができます。

| [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |                  [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})                   |
| :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------: |
|                                         ✔️                                          |                          ❌                          |                                 ❌                                 | [RIP]({{< ref "/docs/defenses/isolation-policies/resource-isolation" >}}) {{< katex>}}^{1}{{< /katex >}} |

____

1. resource isolation policyはエラーベースのcross-site leaksを防ぐのに十分な有効なはずですが、[Framing Isolation Policy]({{< ref "/docs/defenses/isolation-policies/framing-isolation" >}})がないシナリオでは、iframeを通してエラーイベントが漏洩する可能性があります。

## 実例

特定のユーザしかアクセスできないTwitter APIのエンドポイントを悪用できるバグがありました。
このエンドポイントは、所有者以外のすべてのTwitterユーザに対してエラーを返します。
攻撃者はこの挙動を悪用して、ユーザを非匿名化させる可能性があります。
同様に、別のバグではプライベートメッセージの画像認証機構を悪用して、同じ目的を達成することができました[^2] [^1]。

## 参考文献

[^1]: Leaky Images: Targeted Privacy Attacks in the Web, [link](https://www.usenix.org/system/files/sec19fall_staicu_prepub.pdf)
[^2]: Tracking of users on third-party websites using the Twitter cookie, due to a flaw in authenticating image requests, [link](https://hackerone.com/reports/329957)
[^3]: Twitter ID exposure via error-based side-channel attack, [link](https://hackerone.com/reports/505424)
[^4]: Cross-Origin State Inference (COSI) Attacks: Leaking Web Site States through XS-Leaks, [link](https://arxiv.org/pdf/1908.02204.pdf) (see page 6)
