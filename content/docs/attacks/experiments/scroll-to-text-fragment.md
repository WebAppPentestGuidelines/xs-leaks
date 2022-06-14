+++
title = "Scroll to Text Fragment"
description = ""
date = "2020-10-01"
category = "Experiments"
abuse = [
    "onblur",
    "focus",
    "iframes",
]
defenses = [ "Document Policies" ]
menu = "main"
+++

Scroll to Text Fragment (STTF) は、ユーザーがウェブページのテキストの任意の部分へのリンクを作成できるWebプラットフォームの新機能です。フラグメント`#:~:text=`は、ハイライトされ、ブラウザによってビューポートにもたらされるテキストスニペットを運びます。この機能は、攻撃者がこの動作が発生したときに検出することができれば、新しいXS-Leakを導入することができます。この問題は、[Scroll to CSS Selector](https://docs.google.com/document/d/15HVLD6nddA0OaI8Dd0ayBP2jlGw5JpRD-njAyY1oNZo/edit#heading=h.wds2qckm3kh5)のXS-Leakに非常によく似ています。

## 期待される課題・議論される課題

この機能の仕様に関する初期の議論では、単純な実装でいくつかのXS-Leakが導入できることが示されました[^1]。この仕様では、様々な攻撃シナリオ[^3]が考慮されており、Googleの研究結果も同様です[^4]。この機能を実装する際、ブラウザが注意すべきXS-Leakの可能性の1つは、以下の通りです。
In early discussions regarding the specification of this feature it was shown that several XS-Leaks could be introduced with a naïve implementation [^1]. The specification considers various attack scenarios [^3], as does research from Google [^4]. One possible XS-Leak browsers need to be aware of when implementing this feature is:

- 攻撃者は、ページを`iframe`として埋め込むことで、親ドキュメントの`onblur`イベントを聞くことによって、ページがテキストにスクロールされたかどうかを検出することができます。この方法は、[ID Attribute XS-Leak]({{< ref "id-attribute.md" >}})と類似しています。このシナリオは、Chromeの実装[^5]では、トップレベルのナビゲーションで発生するフラグメントナビゲーションのみを許可しているため、緩和されています。

## 現在の課題

{{< hint warning >}}
これらのXS-Leaksは、ターゲットページに何らかのマークアップを注入する必要があります。
{{< /hint >}}

STTFの開発過程で、フラグメントナビゲーションを検出するための新しい攻撃やトリックが発見されました。そのうちのいくつかは今でも有効です。

- 攻撃者が制御する`iframe`を埋め込んだWebページでは、テキストへのスクロールが発生したかどうかを攻撃者が判断することができるかもしれません。これは、[`IntersectionObserver`](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)APIを使用して行うことができます[^2] [^3] [^4]。
- ページが[レイジーローディング](https://web.dev/native-lazy-loading/)で画像を含む場合、攻撃者は画像が[ブラウザにキャッシュ][cached in the browser]({{< ref "../cache-probing.md" >}})されているかどうかをチェックすることで、画像を含むフラグメントナビゲーションが発生したかどうかを検出することができます。これは、[レイジーローディング](https://web.dev/native-lazy-loading/)の画像は、ビューポートに表示されたときにのみ取得（およびキャッシュ）されるため、機能します。

{{< hint important >}}
Scroll to Text Fragmentは、Chromeでのみ利用可能です。仕様書[ドラフト](https://wicg.github.io/scroll-to-text-fragment/)は現在検討中です。
{{< /hint >}}

{{< hint info >}}
Scroll to Text Fragment XS-Leaksでは、ページ上に1つの単語が存在するかどうかを観察し、ユーザーがページに対して何らかの操作（例：マウスクリック）を行った場合にのみ、攻撃者は一度に1ビットの情報を抽出することが可能です。
{{< /hint >}}

## なぜ問題なのか？

攻撃者はSTTFを悪用して、Webページに表示されるユーザーの個人情報を漏えいさせることができます。

### 事例シナリオ

あるユーザが国民健康保険制度のWebサイトにログインしており、そこでユーザの過去の病気や健康問題についての情報にアクセスすることが可能でした。攻撃者はユーザーをそのページの一つに誘い込み、STTFを使用してユーザーの健康状態の詳細を推測できる可能性があります。例えば、ある病名で検索した際に、ページスクロールを検知すると、その病名が被害者の病気であることを知ることができます。


## 対策

| [Document Policies]({{< ref "/docs/defenses/opt-in/document-policies.md" >}}) | [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |                                          [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})                                          |
| :--------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------: |
|                                         ✔️                                          |                                         ✔️                                          |                          ❌                          |                                 ✔️                                 | [RIP]({{< ref "/docs/defenses/isolation-policies/resource-isolation" >}}) 🔗 [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |

## 参考文献

[^1]: Privacy concerns with proposal through inducing network requests, [link](https://github.com/WICG/scroll-to-text-fragment/issues/76)
[^2]: Possible side-channel information leak using IntersectionObserver, [link](https://github.com/WICG/scroll-to-text-fragment/issues/79)
[^3]: Text Fragments - Security and Privacy, [link](https://wicg.github.io/scroll-to-text-fragment/#security-and-privacy)
[^4]: Scroll-to-text Fragment Navigation - Security Issues, [link](https://docs.google.com/document/d/1YHcl1-vE_ZnZ0kL2almeikAj2gkwCq8_5xwIae7PVik/edit#)
[^5]: Boldly link where no one has linked before: Text Fragments, [link](https://web.dev/text-fragments/#privacy)
