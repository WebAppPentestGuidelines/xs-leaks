+++
title = "CORB Leaks"
description = ""
date = "2020-10-01"
category = "Attack"
abuse = [
    "Browser Feature",
    "Error Events",
    "Content-Type",
    "nosniff",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
]
menu = "main"
weight = 2
+++

[Cross-Origin Read Blocking]({{< ref "/docs/defenses/secure-defaults/corb.md" >}}) (CORB) は、Spectre などの投機的サイドチャネル攻撃の影響を軽減することを目的とした、Web プラットフォームのセキュリティ機能です。残念ながら、特定のタイプのリクエストをブロックすることで、あるリクエストではCORBが実行され、別のリクエストでは実行されなかったことを攻撃者が検出できる、新しいタイプのXS-Leaks [^1] が導入されました。とはいえ、導入されたXS-Leaksは、CORBによって積極的に保護される問題(Spectreなど)よりもはるかに影響は少ないです。

{{< hint info >}}
これはChromiumの既知の問題であり、[未修正のまま]((https://docs.google.com/document/d/1kdqstoT1uH5JafGmRXrtKE4yVfjUVmXitjcvJ4tbBvM/edit?ts=5f2c8004))であるかもしれませんが、Chromiumベースのブラウザで[デフォルトでSameSite Cookieが展開される](https://blog.chromium.org/2020/05/resuming-samesite-cookie-changes-in-july.html)ことにより、その影響は大きく軽減されます。
{{< /hint >}}

## CORB & Error Events

攻撃者は、CORBがレスポンスからボディとヘッダーを取り除く結果となるステータス コード`2xx`で、レスポンスが*CORB protected*`Content-Type`（および`nosniff`）を返す場合、CORBの保護機能が強制的に実行されたことを観察することができます。この保護機能を検出すると、攻撃者はステータスコード (成功 もしくは エラー) と `Content-Type` (CORBで保護されているかどうか) の両方の組み合わせを漏らすことができます。これにより、以下の例に示すように、2つの可能な状態を区別することができます。
* 1番目の状態はリクエストがCORBによって保護され、2番目の状態では、クライアントエラー（404）となる。
* 1番目の状態はCORBによって保護され、2番目の状態では保護されない。

以下の手順で、最初の例の文脈でこの保護機能を悪用することができます。

1. 攻撃者は、`Content-Type`が`text/html`で`nosniff` ヘッダーが設定された`200 OK`のレスポンスを返すリソースを`script`タグに、クロスオリジンリソースとして埋め込むこみます。
2. 機密性の高いコンテンツが攻撃者のプロセスに入るのを防ぐため、CORBは元のレスポンスを空のレスポンスに置き換えます。
3. 空のレスポンスは有効なJavaScriptであるため、`onerror` イベントは発生せず、`onload` が代わりに発生します。
4. 攻撃者は、1.と同様に2番目のリクエスト（2番目の状態に対応）をトリガーし、`200 OK` 以外のものを返します。このとき、`onerror` イベントが発生します。

興味深い動作は、CORBがリクエストから有効なリソースを作成し、JavaScript以外を含む可能性がある（エラーを引き起こす）ことです。非CORB環境を考慮すると、1.と 4.の両方のリクエストがエラーを引き起こします。これは、これらの状況によって区別可能であるとしてXS-Leakを導入しています。

## `nosniff`ヘッダーの検出

CORBは、リクエストに `nosniff` ヘッダーが存在する場合、攻撃者に検出させることも可能です。この問題は、CORBがこのヘッダーの存在と一部のスニッフィングアルゴリズムによってのみ強制的に実行されることに起因しています。以下の例では、2つの区別可能な状態を示しています。

1. CORBは、リソースが`nosniff`ヘッダーと共に`Content-Type`が`text/html`で提供される場合、`script`として認識されたリソースを埋め込んだ攻撃者ページを防止します。
2. リソースが`nosniff`を設定せず、CORBがページの `Content-Type`を[推測できない場合](https://chromium.googlesource.com/chromium/src/+/master/services/network/cross_origin_read_blocking_explainer.md#what-types-of-content-are-protected-by-corb)（`text/html`のまま）、コンテンツが有効なJavaScriptとして解析できないため`SyntaxError`が発生します。このエラーは、`script`タグが[特定の条件](https://developer.mozilla.org/en-US/docs/Web/API/HTMLScriptElement)下でのみエラーイベントをトリガーするため、`window.onerror` をリッスンすることで捕捉できます。

## 対策


| [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |                                          [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})                                          |
| :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------: |
|                                         ✔️                                          |                          ❌                          |                                 ❌                                 | [RIP]({{< ref "/docs/defenses/isolation-policies/resource-isolation" >}}) 🔗 [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |

🔗 – 異なるシナリオに対して有効な防御機構を組み合わせる必要があります。

{{< hint tip >}}
開発者は、アプリケーションのサブリソースに[CORP]({{< ref "/docs/defenses/opt-in/corp.md" >}})を展開し、いつ行動するかを決定するためにレスポンスを検査しないCORBと同様の保護を強制することができます。攻撃者がこのXS-Leakを悪用するのを防ぐために、一般的なXS-Leakの防御メカニズムも有効です。
{{< /hint >}}

## 参考文献

[^1]: CORB vs side channels, [link](https://docs.google.com/document/d/1kdqstoT1uH5JafGmRXrtKE4yVfjUVmXitjcvJ4tbBvM/edit?ts=5f2c8004)
