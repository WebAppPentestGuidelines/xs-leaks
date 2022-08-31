+++
title = "SameSite Cookies"
description = ""
date = "2020-10-01"
category = [
    "Defense",
]
menu = "main"
+++

SameSite Cookieは、クロスサイトリクエストを含むセキュリティ問題を修正するための、最もインパクトのある最新のセキュリティ機構の一つです。この機構によって、アプリケーションはブラウザに、同じサイト [^1]で発行されたリクエストにのみCookieを含めるように強制することができます。
SameSite Cookieには、`None`、`Lax`、`Strict` の 3 つのモードがあります。

## SameSite Cookieのモード
SameSite Cookieのモードは以下の通りです。

* `None` – SameSite Cookieによるすべての保護機能を無効化し、Cookieの旧来の動作に戻します。このモードは推奨されません。

{{< hint important >}}
`None` 属性を設定するには、そのCookieに `Secure` 属性を付与しなければなりません。 [^1]
{{< /hint >}}


* `Strict` – ブラウザがクロスサイトリクエストにCookieを含めないようにします。これは、 `<script src="example.com/resource">`, `<img src="example.com/resource">`, `fetch()`、や `XHR` が元になるリクエストには `Strict` モードのCookieを付けずに送信します。 たとえユーザが `example.com/resource` のリンクをクリックしたとしてもそのリクエストにCookieは含まれません。

* `Lax` – `Lax`と`Strict`の唯一の違いは、`Lax`モードではトップレベルの遷移によって発生するクロスサイトリクエストにはCookieを付けることができるということです。`Lax` モードはアプリケーションへの導線リンクを破壊しないため、設定がより簡単になります。残念ながら、攻撃者は`window.open` を通じてトップレベルの遷移を引き起こすことができ、それによって攻撃者は`window`オブジェクトへの参照を維持することができます。

## 考察


`Strict`モードのCookieは最強のセキュリティ保護を提供しますが、既存のアプリケーションに`Strict`モードのSameSite Cookieを設定することは難しいでしょう。

SameSite Cookieは防弾でもなければ [^2]、すべてを解決できる技術ではありません。XS-Leaksに対するこの防御戦略を完全なものにするために、アプリケーションは他の追加的な防御を実装することを検討するべきです。例えば、[COOP]({{< ref "coop.md" >}}) は `Lax` モードの SameSite Cookieが使われたとしても、最初のナビゲーション以降、攻撃者が `window` 参照を使ってページをコントロールするのを防ぐことができます。

{{< hint important >}}
ブラウザによっては、Cookieのデフォルト動作として`Lax`モードを使用しない場合があるので、SameSite属性を明示的に設定し、確実に実行されるようにしてください。
Google Chromeのデフォルトでは、`SameSite`属性を持たないCookieは`Lax`モードがデフォルトの動作になります。しかし、POSTリクエストで送信される際、設定されてから2分未満のCookieについては例外的にクロスサイトであってもCookieが付与されます。[^3]

[^3]: Cookies default to SameSite=Lax, [link](https://www.chromestatus.com/feature/5088147346030592)
{{< /hint >}}

## 実装
この機構をWebアプリケーションに導入することに興味がある場合は [web.dev](https://web.dev/samesite-cookie-recipes/) の記事を見てください。

## 参考資料

[^1]: SameSite cookies explained, [link](https://web.dev/samesite-cookies-explained/)
[^2]: Bypass SameSite Cookies Default to Lax and get CSRF, [link](https://medium.com/@renwa/bypass-samesite-cookies-default-to-lax-and-get-csrf-343ba09b9f2b)
