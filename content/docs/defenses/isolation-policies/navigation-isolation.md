+++
title = "Navigation Isolation Policy"
description = ""
date = "2020-11-30"
category = [
    "Defense",
]
menu = "main"
weight = 3
+++
Navigation Isolation Policy は、クロスサイトウィンドウコンテキストを利用した CSRFやクリックジャッキング、反射型XSS、XS-Leak を緩和することを目的としたサーバーサイドの保護機構です。これは厳格なポリシーであり、ハイパーリンクを介した遷移を含むすべてのクロスサイトの遷移をブロックするため、アプリケーションを破壊する可能性があります。

{{< hint tip >}}
すべてのクロスサイトインタラクションを拒否する代わりに、ユーザーにアクションを確認するよう促すことができます。例えば、*このページを信頼できる発信元から訪れたことを確認することで* 、バックグラウンドでの攻撃のリスクを軽減し、同時にアプリケーションの意図しない破壊を防ぐのに役立ちます。
{{< /hint >}}

## Fetch Metadataを用いた実装

以下のコードは、[Fetch Metadata]({{< ref "../opt-in/fetch-metadata.md">}}) ヘッダーを使用した Navigation Isolation Policy の実装例を示しています[^secmetadata]:

```py
# クロスサイトリクエストを拒否し、クリックジャッキングやXS-Leaks、他のバグから保護します
def allow_request(req):
  # クロスサイトでないリクエストを許可する
  if req['headers']['sec-fetch-site'] != 'cross-site':
    return True

  # ホームページなど、遷移されることを意図したエンドポイントへのリクエストを許可する
  if req.path in whitelisted_paths:
    return True

  # 埋め込みを含むすべてのトップレベルクロスサイトの遷移をブロックする
  if req['headers']['sec-fetch-mode'] in ('navigate', 'nested-navigate'):
      return False

  # その他のリクエストを許可する
  return True
```
## 参考文献
[^secmetadata]: Fetch Metadata Request Headers playground, [link](https://secmetadata.appspot.com/)
