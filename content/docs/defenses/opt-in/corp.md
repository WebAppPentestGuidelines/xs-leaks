+++
title = "Cross-Origin-Resource-Policy"
description = ""
date = "2020-10-01"
category = [
    "Defense",
]
menu = "main"
+++

Cross-Origin Resource Policy（CORP）は、Webプラットフォームのセキュリティ機能であり、Webサイトが特定のリソースを他のオリジンによってロードされるのを防ぐことができます。
この保護はオプトイン防御であるためCORBを補完しますが、CORBはデフォルトで一部のクロスオリジン読み取りをブロックします。
CORPは、開発者が攻撃者によって制御されるプロセスに機密リソースが到達しないようにすることで、投機的実行攻撃とXSリークの両方から保護するように設計されています。
CORBとは異なり、この保護は、アプリケーションが保護をオプトインした場合にのみブラウザーに適用されます。
アプリケーションは、どのオリジンのグループ('same-site','same-origin', 'cross-site')がリソースの読み取りを許可されるかを定義できます。

アプリケーションが特定のリソースCORPヘッダーを「same-site」または「same-origin」として設定した場合、攻撃者はそのリソースを読み取ることができません。これは非常に強力で、推奨される保護です。

CORPを使用するときは、次の点に注意してください。

* CORPはナビゲーションリクエストに対して保護しません。 つまり、プロセス外のiframeをサポートしていないブラウザでは、CORPで保護されたリソースは、[Framing protections]（{{<ref "../opt-in/xfo.md ">}}）は使用されていない場合、他のオリジンのプロセスで終了する可能性があります。
* CORPを使用すると、[a new XS-Leak]（{{<ref "../../attacks/browser-features/corp.md">}}）も付随し、攻撃者はあるリクエストでCORPが実施されたかどうかを検出することができます。

## 参考文献

[^1]: Cross-Origin Resource Policy (CORP), [link](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cross-Origin_Resource_Policy_(CORP))
