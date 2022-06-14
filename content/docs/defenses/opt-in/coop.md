+++
title = "Cross-Origin-Opener-Policy"
description = ""
date = "2020-10-01"
category = [
    "Defense",
]
menu = "main"
+++

ウェブサイトの `window` オブジェクトにアクセスすることは、さまざまな XS-Leak テクニックにとって共通の前提条件です。
[Framing Protections]({{< ref "xfo.md" >}}) は、攻撃者が `window` オブジェクトにアクセスするために iframe を使用できないようにしますが、これは `window.open(url)` や `window.opener` リファレンスを通して開かれたウィンドウから `window` オブジェクトに攻撃者がアクセスするのを止めることはありません。

XS-Leaks を `window.open` で利用することは、一般に攻撃者にとって最も魅力のない方法と考えられています。なぜなら、ユーザーは開いているブラウザウィンドウでそれが起こるのを見ることができるからです。しかし、次のような場合には、通常、正しい方法です。

- [Framing Protections]({{< ref "xfo.md" >}})がセットされている
- [Same-Site Cookies with `Lax` Mode]({{< ref "same-site-cookies.md" >}}) がセットされている(`Strict` モードとは対照的に，`Lax` モードではトップレベルのウィンドウを移動することができます)


他のウェブサイトがページへの任意のウィンドウ参照を得ることを防ぐために、アプリケーションは Cross-Origin-Opener-Policy (COOP) [^1] [^2] を展開することができます。

COOPヘッダーには3つの値があります。

* `unsafe-none` - これはデフォルトの値で、値が設定されていない場合にウェブサイトがどのように振る舞うかを示しています。
* `same-origin` - これは最も厳しい値です。`same-origin` を設定すると、クロスオリジンのウェブサイトは新しいウィンドウを開いて `window` オブジェクトにアクセスすることができなくなります。もし、アプリケーションが `window.open` を使って別のウェブサイトを開いて通信することに依存している場合、`same-origin` によってブロックされます。これが問題になる場合は、代わりに `same-origin-allow-popups` を設定してください。
* `same-origin-allow-popups` - この値はあなたのウェブサイトが `window.open` を使用することを許可しますが、他のウェブサイトがあなたのアプリケーションに対して `window.open` を使用することは許可されません。

可能であれば、`same-origin`を設定することをお勧めします。`same-origin-allow-popups` を設定した場合は、 `window.open` で開くウェブサイトを確認し、そのウェブサイトが信頼できるものであることを確認するようにしてください。

## Considerations

COOP はオプトインのメカニズムであり、非常に新しいものなので、開発者やセキュリティエンジニアは簡単に見落としてしまいます。
それでも、この防御メカニズムは重要です。
なぜなら、攻撃者が `window.open` のような API が返すウィンドウ参照を利用する XS-Leak を悪用するのを防ぐ唯一の方法だからです（`Strict` モードの SameSite Cookie が広く展開されない限り）。

## Deployment

[web.dev](https://web.dev/why-coop-coep/)の記事で、このプロテクションの利点と導入方法について詳しく説明しています。

## References

[^1]: Cross-Origin-Opener-Policy response header (also known as COOP), [link](https://gist.github.com/annevk/6f2dd8c79c77123f39797f6bdac43f3e)
[^2]: Cross-Origin-Opener-Policy, [link](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Opener-Policy)
