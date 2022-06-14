+++
title = "SameSite Cookies"
description = ""
date = "2020-10-01"
category = [
    "Defense",
]
menu = "main"
+++

SameSite Cookieは、クロスサイトリクエストを含むセキュリティ問題を修正するための最も影響力のある最新のセキュリティメカニズムの1つです。このメカニズムにより、アプリケーションは、同じサイト[^1] で発行されたリクエストにCookieのみをブラウザに含めるように強制できます。

このタイプのCookieには3つのモードがあります。： `None`, `Lax`, `Strict`

## SameSite Cookie Modes

次のSameSite属性値を使用できます:

* `None` – すべての保護を無効にし、Cookieの古い動作を復元します。このモードはお勧めしません。

  {{< hint important >}}
`None` 属性を使うには`Secure`属性を付ける必要があります [^same-site-none].
  [^same-site-none]: Same site属性についての説明 [link](https://web.dev/samesite-cookies-explained/#samesitenone-must-be-secure)
  {{< /hint >}}


* `Strict` – ブラウザがクロスサイトリクエストにCookieを含めないようにします。例えば `<script src="example.com/resource">`, `<img src="example.com/resource">`, `fetch()`や `XHR` 等のリクエストについて、`Strict`なCookieを添付せずに行います. ユーザーが`example.com/resource`へのリンクをクリックしても、Cookieは含まれません。

* `Lax` – `Lax`と`Strict` の唯一の違いは、`Lax` モードでは、クロスサイトのトップレベルナビゲーションによってトリガーされたリクエストにCookieを追加できることです。これにより、アプリケーションへのリンクが壊れないため、Cookieのデプロイがはるかに簡単になります。残念ながら、攻撃者は`window.open` を介してトップレベルのナビゲーションを引き起こすことができ、それによって攻撃者は`window`オブジェクトへの参照を維持することができます。

## Considerations

`Strict` クッキーは最も強力なセキュリティ保証を提供しますが、既存のアプリケーションに `Strict` のSameSite Cookieを配備するのは非常に難しいかもしれません。

SameSite Cookieは根本的な対策 [^2] ではありませんし、すべてを解決することもできません。
XS-Leaks に対するこの防御戦略を補完するために、アプリケーションは他の追加の防御を実装することを検討すべきです。
例えば、[COOP]({{< ref "coop.md" >}}) は `Lax` モードの SameSite クッキーが使われていても、攻撃者が最初のナビゲーション以降に `window` 参照を使ってページをコントロールするのを防ぐことができます。

{{<ヒント important >}}
ブラウザによっては、Laxのデフォルトを使用しない場合があるので、明示的にSameSite属性を設定し、確実に実行されるようにしてください。
デフォルトでは、`SameSite`属性を持たないChromeのクッキーは、`Lax`モードがデフォルトとなります。
しかし、POSTリクエストで送信される2分未満前に設定されたクッキーについては、その動作に例外があります。[^3]

[^3]: Cookies default to SameSite=Lax, [link](https://www.chromestatus.com/feature/5088147346030592)
{{< /hint >}}

## Deployment

このメカニズムをWebアプリケーションにデプロイすることに関心がある人は、この [web.devの記事](https://web.dev/samesite-cookie-recipes/)を読んでください。

## References

[^1]: SameSite cookies explained, [link](https://web.dev/samesite-cookies-explained/)
[^2]: Bypass SameSite Cookies Default to Lax and get CSRF, [link](https://medium.com/@renwa/bypass-samesite-cookies-default-to-lax-and-get-csrf-343ba09b9f2b)
