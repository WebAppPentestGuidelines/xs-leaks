+++
title = "Cross-Origin Read Blocking"
description = ""
date = "2020-10-01"
category = "Defense"
menu = "main"
+++

Cross-Origin Read Blocking (CORB)は、攻撃者が特定のクロスオリジンのリソースを読み込むことを防ぐセキュリティ機構です[^1]。
この保護機構は、攻撃者がクロスサイトのページ（*attacker.com* と *sensitive.com* など）が埋め込まれたプロセスのメモリを読み取れるようにするSpectreなどの投機的なサイドチャネル攻撃から保護するために作成されました。
CORBは、攻撃者が特定の機密性の高いクロスオリジンのリソースを攻撃者が制御するプロセスに読み込むのを防ぐことを目的としています。
たとえば、攻撃者がクロスオリジンのHTML、XML、JSONを `img`タグに読み込もうとしようとすると、CORBはこれを阻止します。
CORBを使用すると、サーバがデータを返さなかったかのように処理されます。

リソースを分類するために、CORBは `Content-Type`ヘッダ、` nosniff`ヘッダ、およびその他のさまざまなヒューリスティックを使用します。

{{< hint info >}}
[Cross-Origin Resource Policy (CORP)]({{< ref "../opt-in/corp.md" >}})は、CORB を適用・拡張するオプトインの保護です。
{{< /hint >}}

CORBを使用するときは、以下の点に注意してください。

* 現在はChromiumベースのブラウザのみがCORBをサポートしています。
* CORBは、ナビゲーショナルリクエストに対して保護しません。つまり、プロセス外のiframeをサポートしないブラウザでは、[framing protections]({{< ref "../opt-in/xfo.md" >}})が使用されていない場合、CORBで保護されたリソースは別のオリジンのプロセスで終了する可能性があります。
* CORBは、攻撃者がCORBの結果を観察できる可能性があるため、[new XS-Leak]({{< ref "../../attacks/browser-features/corb.md" >}}) 手法を引き入れており、これによってさまざまな情報のリークが発生する可能性があります。 ただし、ほとんどの場合、これらのリークは投機的実行攻撃によってリークされる可能性のあるデータよりも影響は小さいものでしょう。

## 参考文献

[^1]: Cross-Origin Read Blocking for Web Developers, [link](https://chromium.googlesource.com/chromium/src/+/master/services/network/cross_origin_read_blocking_explainer.md)
