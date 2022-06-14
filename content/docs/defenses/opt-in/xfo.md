+++
title = "Framing Protections"
description = ""
date = "2020-10-01"
category = [
    "Defense",
]
menu = "main"
+++

かなりの数のXS-Leaksがiframeのプロパティの一部に依存しています。攻撃者がページのコンテンツを`iframe`, `frame`, `embed` または `object`として埋め込むことができない場合、攻撃は不可能になる可能性があります。

これらのオブジェクトに依存するXSリークを軽減するために、ページはそれらを埋め込むことができるオリジンを禁止または選択できます。
このためには、[`X-Frame-Options` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) や [CSP frame-ancestors directive](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)を使います。

Framing Protectionsを適用するWebサイトは攻撃者の発信元から埋め込むことができないため、Webサイトはレンダリングされず、JavaScriptは実行されません。したがって、そのサブリソース（画像、JS、またはCSS）はいずれもブラウザによって取得されません。

{{< hint tip >}}
 [CSP frame-ancestors directive](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors) は 、Framing Protectionsを有効にする最新の方法です。ただし、Internet Explorerではサポートされていないため、多くの場合、ヘッダーと組み合わせて使用​​することをお勧めします。

ただし、Internet Explorerではサポートされていないため、多くの場合、 [`X-Frame-Options` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)ヘッダーと組み合わせて使用​​することをお勧めします。
{{< /hint >}}

## Considerations

この保護は、[rely on framing](../../../../abuse/iframes/)に依存するXSリークに対して非常に効果的であり、アプリケーションの大部分を壊すことなく簡単に実装できます。このメカニズムは、一部のXS-Leaksから保護するだけでなく、 [clickjacking](https://owasp.org/www-community/attacks/Clickjacking)などの攻撃も防ぎます。

## Deployment

多くのアプリケーションはクロスオリジンで`iframe`に埋め込まれることを意図していないため、Framing Protectionsのデプロイは通常簡単です。このヘッダーの利点の詳細については、[web.devの記事](https://web.dev/same-origin-policy/) を確認してください。