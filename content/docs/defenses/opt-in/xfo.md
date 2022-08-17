+++
title = "Framing Protections"
description = ""
date = "2020-10-01"
category = [
    "Defense",
]
menu = "main"
+++

相当数のXS-Leaksがiframeのいくつかの特性に依存しています。もし攻撃者がページのコンテンツを `iframe`、`frame`、`embed` または `object` として埋め込むことができなければ、もはや攻撃は不可能になるかもしれません。これらのコンテンツに依存するXS-Leaksを軽減するために、ページはどのオリジンがそれらを埋め込むことができるかを禁じたり、あるいは選択したりすることができます。これは、[`X-Frame-Options` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) や [CSP frame-ancestors directive](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors) を使用することで可能になります。

Framing Protections を適用したウェブサイトは、攻撃者のオリジンから埋め込むことができないため、コンテンツはレンダリングされず、JavaScript は実行されません。したがって、そのサブリソース（画像、JS、CSSなど）はいずれもブラウザによって取得できません。

{{< hint tip >}}
[CSP frame-ancestors directive](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)は、フレーミング保護を有効にする、より現代的な方法です。しかし、Internet Explorer ではサポートされていないので、多くの場合、[`X-Frame-Options` Header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) と組み合わせて使うことが推奨されます。
{{< /hint >}}

## Considerations

この保護機能は [rely on framing](../../../../abuse/iframes/) のXS-Leaksに対して非常に有効であり、大多数のアプリケーションに対して容易に非破壊で実装することができます。この仕組みは、一部のXS-Leakを防ぐだけでなく、[clickjacking](https://owasp.org/www-community/attacks/Clickjacking)のような攻撃も防ぐことができます。

## Deployment

通常、多くのアプリケーションは `iframe` 内にクロスオリジンで埋め込まれることを意図していないので、フレーミング保護を導入することは簡単です。このヘッダの利点については、この [web.dev](https://web.dev/same-origin-policy/) の記事を参照してください。
