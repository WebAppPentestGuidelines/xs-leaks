+++
title = "Resource Isolation Policy"
description = ""
date = "2020-11-30"
category = [
    "Defense",
]
menu = "main"
weight = 1
+++
Resource Isolation Policyは、外部のWebサイトがリソースを要求するのを防ぎます。このようなトラフィックをブロックすることで、CSRF、XSSI、XS-Leaks などの一般的な Web 脆弱性を軽減できます。このポリシーは、エンドポイントがクロスサイトコンテキストでロードされることを意図していないアプリケーションに対して有効にすることができ、アプリケーションから来るリソース要求だけでなく、直接の遷移も可能にします。

## Fetch Metadataを用いた実装

以下のスニペットは、[Fetch Metadata]({{< ref "../opt-in/fetch-metadata.md">}}) ヘッダーを使用したResource Isolation Policyの実装例を示しています:

```py
# クロスサイトリクエストを拒否し、クリックジャッキングやXS-Leaks、他のバグから保護します
def allow_request(req):
  # [OPTIONAL] クロスオリジンで提供されることを意図したパス/エンドポイントを除外する。
  if req.path in ('/my_CORS_endpoint', '/favicon.png'):
    return True

  # `Cross-Origin-Resource-Policy: same-site`を設定すると安全です。(考慮事項参照)

  # Fetch Metadataを送信しないブラウザからのリクエストを許可する
  if not req['headers']['sec-fetch-site']:
    return True

  # same-siteやブラウザ経由のリクエストの許可
  if req['headers']['sec-fetch-site'] in ('same-origin', 'same-site', 'none'):
    return True

  # 埋め込みを含むシンプルなトップレベルの遷移を許可する
  if req['headers']['sec-fetch-mode'] == 'navigate' and req.method == 'GET':
      return True

  # その他のリクエストを拒否する
  return False
```

## 考慮事項
Resource Isolation Policyから明示的に除外されていないすべてのリクエストに `Cross-Origin-Resource-Policy: same-site` レスポンスヘッダを設定しても問題ないはずです。[CORP]({{< ref "../opt-in/corp.md" >}})を参照してください。

## デプロイメント

[web.dev](https://web.dev/fetch-metadata/)の記事で、この保護機能についての詳細や異なるポリシー、導入方法のヒントをご覧ください。
