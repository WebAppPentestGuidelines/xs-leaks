+++
title = "Strict Isolation Policy"
description = ""
date = "2020-11-30"
category = [
    "Defense",
]
menu = "main"
weight = 4
+++
Strict Isolation Policyは、すべてのクロスサイトインタラクション（ハイパーリンクを介したアプリケーションへの遷移を含む）から保護することを目的としています。これは非常に厳しいポリシーであり、アプリケーションが正常に機能しなくなる可能性があります。

{{< hint tip >}}
すべてのクロスサイトインタラクションを拒否する代わりに、ユーザーにアクションを確認するよう促すことができます。例えば、*このページを信頼できる発信元から訪れたことを確認することで* 、バックグラウンドでの攻撃のリスクを軽減し、同時にアプリケーションの意図しない破損を防ぐのに役立ちます。

しかし、他のリソースはバックグラウンドで読み込まれるため、これは遷移のリクエストに対してのみ機能します。
{{< /hint >}}


## Fetch Metadataを用いた実装

以下のスニペットは、アプリケーションによる Strict Isolation Policy の実装例を示しています。

```py
# クロスオリジンリクエストを拒否し、CSRFやXSSI、他のバグから保護します
def allow_request(req):
  # Fetch Metadataを送信しないブラウザからのリクエストを許可する
  if not req['headers']['sec-fetch-site']:
    return True

  # cross-siteリクエストをブロック
  if req['headers']['sec-fetch-site'] == 'cross-site':
    return False

  # その他のリクエストを許可する
  return True
```

## SameSite cookiesを用いた実装
もしサーバーが[`SameSite=strict`]({{< ref "../opt-in/same-site-cookies/#samesite-cookie-modes" >}})フラグを持つクッキーを送ると、このスニペットで示されるように、そのクッキーを含まない返されたリクエストは拒否されることがあります。

```py
# クロスオリジンリクエストを拒否し、CSRFやXSSI、他のバグから保護します
def allow_request(req):

  if req['cookies']['strict-cookie'] == 'true':
    return True

  # strict Cookieを持たないリクエストはブロック
  return False
```

## Refererを用いた実装
また、[`Referer`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer) ヘッダーを使用して、信頼できない送信元からのリクエストを拒否することも可能です:

```py
# 信頼できないリファラーから来たリクエストを拒否する
def allow_request(req):

  # referer ヘッダが信頼できるかどうかを確認する。つまりtrusted_referers ディクショナリに存在するかどうかを確認する。
  if req['headers']['referer'] in trusted_referers:
    return True

  # strict Cookieを持たないリクエストをブロック
  return False
```

{{< hint important >}}
すべてのリクエストに Referer ヘッダが含まれることは保証されていません (たとえば、拡張機能でヘッダを取り除くことができます)。また、`Referer` の値を `null` に設定することも可能であることに注意してください。

Twitterは、XS-Leaksに対して同様の防御策を導入[^twitter_silhouette]しています。

[^twitter_silhouette]: Silhouetteからユーザーアイデンティティを保護する, [link](https://blog.twitter.com/engineering/en_us/topics/insights/2018/twitter_silhouette.html)
{{< /hint >}}
