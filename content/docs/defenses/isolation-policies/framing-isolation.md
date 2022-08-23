+++
title = "Framing Isolation Policy"
description = ""
date = "2020-11-30"
category = [
    "Defense",
]
menu = "main"
weight = 2
+++
Framing Isolation Policy は、[Framing Protections]({{< ref "../opt-in/xfo" >}}) のより厳格なバージョンで、ブラウザではなく、アプリケーションレベルでリクエストがブロックされます。これは、フレーム化を意図していないエンドポイントへのフレーム化リクエストをブロックすることで、さまざまな攻撃（XSSI、CSRF、XS-Leaksなど）から保護するために設計されています。

[Resource Isolation Policy]({{< ref "resource-isolation.md" >}}) と組み合わせることで、クロスサイト情報漏えいの攻撃対象領域を効果的に絞り込むことができます。

{{< hint tip >}}
フレーム化できないエンドポイントをすべて拒否するのではなく、例えば *信頼できる発信元からこのページを訪問したことを確認する* といったアクションを確認するようユーザーに促すことで、バックグラウンドでの攻撃のリスクを軽減し、同時にアプリケーションの意図しない破損の防止に役立てることができます。
{{< /hint >}}

{{< hint tip >}}
[Resource Isolation Policy]({{< ref "resource-isolation.md" >}}) と共に展開された場合、Framing Isolation Policy はウィンドウ参照（例 `window.length`）を利用したリークから保護しないため、[COOP]({{< ref "../opt-in/coop" >}}) や [Navigation Isolation Policy]({{< ref "navigation-isolation" >}}) など他の画面遷移保護が役に立つことがあります。
{{< /hint >}}

## Fetch Metadataを用いた実装

以下のスニペットは、アプリケーションによる Framing Isolation Policy の実装例を示しています。

```py
# CSRF、XSSI、XS-Leaksなどのバグから保護するために、クロスサイトリクエストを拒否します。
def allow_request(req):
  # Fetch Metadataを送信しないブラウザからのリクエストを許可する。
  if not req['headers']['sec-fetch-site']:
    return True
  if not req['headers']['sec-fetch-mode']:
    return True
  if not req['headers']['sec-fetch-dest']:
    return True

  # 遷移以外のリクエストを許可する。
  if req['headers']['sec-fetch-mode'] not in ('navigate', 'nested-navigate'):
    return True

  # 遷移以外のリクエストを許可する。
  if req['headers']['sec-fetch-dest'] not in ('frame', 'iframe', 'embed', 'object'):
      return True

  # [OPTIONAL] クロスサイトで提供されることを意図したパス/エンドポイントを除外する。
  if req.path in ('/my_frame_ancestors_host_src'):
    return True

  # 全ての他のリクエストを拒否する。
  return False
```

## 考慮事項
エンドポイントが`X-Frame-Options` および/または Content Security Policy の `frame-ancestors` ディレクティブによって特定のオリジンからのフレーミングリクエストを許可する場合、Framing Isolation Policy を適用することはできません。
