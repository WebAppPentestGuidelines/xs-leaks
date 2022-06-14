+++
title = "Portals"
description = ""
date = "2020-10-01"
category = "Experiments"
menu = "main"
+++

[Portals](https://web.dev/hands-on-portals/) は、`iframe`に似ていますが、スピードとユーザーエクスペリエンスをより重視したウェブの新しい機能です。[`portal`](https://web.dev/hands-on-portals/)要素は、Chromiumベースのブラウザーで、プリファレンス・フラグの下でのみ利用可能です。対応する[仕様](https://wicg.github.io/portals/)は現在も活発に議論されています。 

残念ながら、この新機能の研究により、新たなXS-Leaksを含むいくつかの重大な問題が発見されました[^2]。

## ID Leaks

Portalsは、[ID Attribute XS-Leak]({{< ref "../id-attribute.md" >}})[framing protections]({{< ref "../../defenses/opt-in/xfo.md" >}})を設定している場合、`Portals` 要素を代わりに使って同じ手法を適用することができます[^1]。

## 参考文献

[^1]: Detecting IDs using Portal, [link](https://portswigger.net/research/xs-leak-detecting-ids-using-portal)
[^2]: Security analysis of \<portal\> element, [link](https://research.securitum.com/security-analysis-of-portal-element/)
