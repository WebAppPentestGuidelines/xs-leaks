+++
title = "CORP Leaks"
description = ""
date = "2020-10-01"
category = "Attack"
abuse = [
    "Browser Feature",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
]
menu = "main"
weight = 2
+++

## 説明

[Cross-Origin Resource Policy]({{< ref "/docs/defenses/opt-in/corp.md" >}}) (CORP) は、Webプラットフォームセキュリティ機能で、Webサイトが特定のリソースを他のオリジンから読み込むことを防止することができます。[CORB]({{< ref "/docs/defenses/secure-defaults/corb.md" >}})がデフォルトで一部のクロスオリジン読み込みをブロックするのに対し、CORPはオプトインの防御であるため、この保護機能はCORBを補完するものです。残念ながら、[CORB]({{< ref "corb.md" >}})と同様に、アプリケーションがこの保護の使用を誤って設定すると、新しいXS-Leakをもたらす可能性があります。

`CORP`がユーザーデータに基づいて強制される場合、WebページはXS-Leakをもたらすことになります。ページ検索機能が、結果を表示するときは`CORP`を強制し、結果を返さないときは強制しないのであれば、攻撃者は2つのシナリオを区別することができます。これは、`CORP`によって保護されたページ/リソースがクロスオリジンでフェッチされるとエラーを返すために発生します。

## 対策

アプリケーションは、CORPがすべてのアプリケーションリソース/エンドポイントに展開されていることを保証すれば、このXS-Leakを回避することができます。さらに、クロスサイトリクエストの無効化を可能にする一般的なセキュリティメカニズムも、この攻撃を防ぐのに役立ちます。

| [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |                                          [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})                                          |
| :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------: |
|                                         ✔️                                          |                          ❌                          |                                 ❌                                 | [RIP]({{< ref "/docs/defenses/isolation-policies/resource-isolation" >}}) 🔗 [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |

🔗 – 異なるシナリオに対して有効な防御機構を組み合わせる必要があります。

