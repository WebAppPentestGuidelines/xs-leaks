+++
title = "CSS Injection"
category = [
    "Attack"
]
abuse = [
    "CSS"
]
menu = "main"
weight = 11
+++

## CSSインジェクション

{{< hint warning >}}
ここで紹介される一連のXS-Leaksは対象ページでのCSSインジェクションを必要とします。
{{< /hint >}}

CSSインジェクションの様々なベクトルの中で、最もよく見られるのは、CSSセレクタの悪用です。これらは、特定のHTML要素にマッチし、選択するための表現として使用することができます。例えば、セレクタ `input[value^="a"]` は、`input` タグの値が文字 "a" で始まっている場合にマッチングされます。したがって、ある CSS セレクタが表現にマッチするかどうかを検出するため、攻撃者は `background` や `@import` などのプロパティを使用して、自身の管理するウェブサイトへのコールバックをトリガーすることができます。[^1] [^2] このマッチングプロセスは簡単にブルートフォースされ、文字列全体に拡張できます。

文字の並びに独自の表記がある場合、フォントの[合字](https://wikipedia.org/wiki/Ligature_(writing))を悪用して、ページに含まれるJavaScriptなどがリークされる可能性があります。

`* { display: block; }`のようなスタイルを適用すると、`style`や`script`などの通常は隠されているHTMLタグも、テキストとして表示されることがあります。したがって、それらの内容もリークされる可能性があります。

## 対策

- 攻撃者がコントロールするコンテンツを独立したドキュメントに置きます。これは srcdoc属性を持つiframeを使って行うことができます。オプションで、コンテンツを独自のオリジンに分離するためのサンドボックス属性を含めることができます。
- CSSインライナーを使って、グローバルなスタイルが変換されるようにします。

| [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |                  [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})                   |
| :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------: |
|                                         ❌                                          |                          ❌                          |                                 ❌                                 | ❌  |
## References
[^1]: CSS Injection Primitives, [link](https://x-c3ll.github.io/posts/CSS-Injection-Primitives/)
[^2]: HTTPLeaks, [link](https://github.com/cure53/HTTPLeaks/)
[^3]: Font ligatures, [link](https://sekurak.pl/wykradanie-danych-w-swietnym-stylu-czyli-jak-wykorzystac-css-y-do-atakow-na-webaplikacje/)
