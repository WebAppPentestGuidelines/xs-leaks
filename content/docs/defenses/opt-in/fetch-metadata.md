+++
title = "Fetch Metadata"
description = ""
date = "2020-11-30"
category = [
    "Defense",
]
menu = "main"
+++

[Fetch Metadata Request Headers](https://www.w3.org/TR/fetch-metadata/) は、ブラウザが HTTPS リクエストで送信するヘッダです。これらのヘッダは、リクエストがどのように発生したかについてのコンテキストを提供し、アプリケーション側がそれに対してどのように応答するかについて、より多くの情報に基づいた決定を行うことができるようにします。これにより、サーバーは潜在的な攻撃（例：予期しないクロスオリジンリクエスト）を検知した際に、異なる動作をすることができます[^1]。これは、サーバーに厳格なポリシーが導入されていれば、XSSI、XS-Leaks、クリックジャッキング、CSRF などのクロスオリジン攻撃に対して非常に効果的です。

XS-Leaksのシナリオでは、サーバーはリクエストがいつクロスオリジン（例：攻撃者のオリジン）で行われたかを知る能力を持ち、ユーザーデータを含まないような別のレスポンスを返すことができます。この種のレスポンスは、ユーザーに関するいかなる情報または状態も提供しないため、攻撃者にとって有用ではなくなります。Fetch Metadataはまた、フレーミングや遷移のリクエストをブロックするために使用することができます。

{{< hint important >}}
セキュリティ上の理由から、Fetch Metadataヘッダは暗号化されたリクエスト（HTTPS）にのみ付与されます。
{{< /hint >}}

## Fetch Metadata vs. SameSite cookie

Fetch Metadata ヘッダは、SameSite Cookieによって提供される保護を拡張するために使用することができます。Fetch Metadata ヘッダと SameSite Cookieの両方がクロスサイトリクエストを拒否するために使われることがありますが Fetch Metadata は以下のような要素に基づいて、より情報に基づいた決定をすることができます。
* リクエストはSame Origin（同一生成元）か、Same Site（同一サイト）か
* リクエストはどのように開始されたか(例: fetch、スクリプト、トップナビゲーション)
* リクエストはユーザの操作によって発生したのか
* リクエストはブラウザによって発生したか（例：オムニボックスに直接URLを入力する）

これは、SameSite Cookieがサービスの機能を壊す可能性があるシナリオにおいて、より正確な保護の展開を可能にします。SameSite Cookieと比較した Fetch Metadata の欠点は、前者が暗号化されていないリクエスト(HTTP)も保護できるのに対し、後者がそうできないことです。

## 考察

Fetch Metadataヘッダは徹底した防護戦略のための有用なツールですが、 [SameSite Cookies]({{< ref "same-site-cookies.md" >}}), [COOP]({{< ref "coop.md" >}}), あるいは [Framing Protections]({{< ref "xfo.md" >}}) といった仕組みの代わりとして見なされるものではありません。たとえFetch Metadataヘッダを使用して同様の結果を得ることができるとしても、サーバに加えてクライアント側でもこれらの制限を実施することが最善です。

Fetch Metadataヘッダの有用性は、アプリケーションの対象範囲とデプロイの正しさに依存します。

## ポリシー

Fetch Metadata Request Headers を利用した特定のポリシーについては [Resource Isolation Policy]({{< ref "../isolation-policies/resource-isolation.md" >}}) と [Framing Isolation Policy]({{< ref "../isolation-policies/framing-isolation.md" >}} を参照してください。
