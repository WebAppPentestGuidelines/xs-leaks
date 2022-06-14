+++
title = "Connection Pool"
description = ""
date = "2020-10-01"
category = "Attack"
abuse = [
    "Connection Pool",
    "Browser Limits",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
]
menu = "main"
+++

network timingを測定する方法の一つとして、ブラウザのソケットプールを悪用する方法があります。
ブラウザはサーバと通信するために、ソケットを利用します。
ハードウェアやその上で動作するOSのリソースには限りがあるため、ブラウザにも制限をかけざるを得ません。

この制限の存在を悪用するために、攻撃者は下記のようなことができます。

1. ブラウザの制限を確認する。
2. 単に接続をハングアップさせる{{< katex>}}255{{< /katex >}}のリクエストを別々のホストに実行して、長時間{{< katex>}}255{{< /katex >}}のソケットをブロックする。
3. ターゲットページに対するリクエストを実行して、{{< katex>}}256^{番目}{{< /katex >}}のソケットを利用する。
4. 他のホストへの {{< katex>}}257^{番目}{{< /katex >}}のリクエストを実行します。(step2、3で)すべてのソケットが使用されているので、このリクエストは、プールが利用可能なソケットを受け取るまで待機する必要があります。この待ち時間は、ターゲットページに属する {{< katex>}}256^{番目}{{< /katex >}}のソケットのネットワークタイミングを、攻撃者に提供します。これが動作するのは、ステップ2の{{< katex>}}255{{< /katex >}}個のソケットがまだブロックされているので、ステップ3のソケットの解放によってプールが利用可能なソケットを受信した場合です。{{< katex>}}256^{番目}{{< /katex >}} のソケットを解放する時間は、リクエストを完了するのにかかった時間と直結しています。

## 対策

| [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) | [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}}) |
| :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :-------------------------------------------------------------------: |
|                                         ❌                                          |                          ❌                          |                                 ❌                                 |                                   ❌                                   |


{{< hint info >}}
[partitioned caches]({{< ref "../../defenses/secure-defaults/partitioned-cache.md" >}})と同様に、リソースの"site/originごとの分割"の原理を[ソケットプール](https://bugzilla.mozilla.org/show_bug.cgi?id=1572544)に拡張することを、いくつかのブラウザが検討しています。
{{< /hint >}}

## 参考文献

[^1]: Leak cross-window request timing by exhausting connection pool, [link](https://bugs.chromium.org/p/chromium/issues/detail?id=843157)
