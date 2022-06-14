+++
title = "Fetch Metadata"
description = ""
date = "2020-11-30"
category = [
    "Defense",
]
menu = "main"
+++

[Fetch Metadata Request Headers](https://www.w3.org/TR/fetch-metadata/) は、HTTPSリクエストを使用してブラウザから送信されます。これらのヘッダーは、リクエストがどのように開始されたかに関するコンテキストを提供し、アプリケーションがそれらに応答する方法についてより多くの情報に基づいた決定を下せるようにします。これにより、サーバーが潜在的な攻撃（予期しないクロスオリジンリクエストなど）を検出したときに、動作を変更して攻撃を緩和することができます[^1]。これは、サーバーに厳密なポリシーが導入されている場合、XSSI、XSリーク、クリックジャッキング、CSRFなどのクロスオリジン攻撃に対して非常に効果的です。

XS-Leaksシナリオでは、サーバは、リクエストがクロスオリジン（攻撃者のオリジンなど）で行われたことを知ることができ、ユーザデータなしで別の応答を返すことができます。この種の応答は、ユーザに関する情報や状態を伝達しないため、攻撃者にとっては役に立ちません。フェッチメタデータを使用して、フレーミングやナビゲーションリクエストをブロックすることもできます。

{{< hint important >}}
セキュリティ上の理由から、Fetchメタデータヘッダーは暗号化された（HTTPS）リクエストにのみ添付されます。
{{< /hint >}}

## Fetch Metadata vs. SameSite cookies

Fetch Metadata ヘッダは、SameSite クッキーによって提供される保護を拡張するために使用することができます。
Fetch Metadata ヘッダと SameSite クッキーは両方ともクロスサイトリクエストを拒絶するために使うことができますが、Fetch Metadata は次のような要素に基づいてより多くの情報に基づいた決定をすることができます。

* リクエストは same-origin か same-site か？
* リクエストはどのように開始されたのか？（例：フェッチ、スクリプト、トップナビゲーション）。
* リクエストはユーザーとの対話によって開始されたのか？
* リクエストはブラウザによって開始されたか（例：オムニボックスに直接URLを入力した）？

これは、SameSite クッキーがサービスの機能性を壊す可能性があるシナリオにおいて、より正確な保護の展開を可能にします。
Fetch Metadata の欠点は、SameSite クッキーであれば暗号化されていないリクエスト (HTTP) においても保護出来るのに対して、HTTPSでしか保護できない点です。

## Considerations

Fetch Metadataヘッダの取得は多層防御戦略に役立つツールですが、[SameSite Cookies]({{< ref "same-site-cookies.md" >}}), [COOP]({{< ref "coop.md" >}}), または [Framing Protections]({{< ref "xfo.md" >}})などのメカニズムの代わりと見なすべきではありません。
Fetch Metadataヘッダを使用して同様の結果を得ることができますが、サーバだけでなくクライアント側でもこれらの制限を適用することをお勧めします。

Fetch Metadataヘッダの有用性は、アプリケーションのカバレッジとデプロイメントの正確さに依存します。

## Policies

フェッチメタデータ要求ヘッダーを利用する特定のポリシーについては、[Resource Isolation Policy]({{< ref "../isolation-policies/resource-isolation.md" >}}) および [Framing Isolation Policy]({{< ref "../isolation-policies/framing-isolation.md" >}}) を参照してください。
