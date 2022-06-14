+++
title = "Partitioned HTTP Cache"
description = ""
date = "2020-10-01"
category = [
    "Defense",
]
menu = "main"
+++

cache probing攻撃を対策するために、ブラウザ開発者は、各Webサイトが個別のキャッシュを持つようにするパーティション化されたHTTPキャッシュ機能の実装に積極的に取り組んでいます。
cache probingは、ブラウザのHTTPキャッシュがすべてのWebサイトで共有されるという事実に依存しているため、パーティション化されたHTTPキャッシュは、多くのcache probingの手法を対策できます。
これは、キャッシュが要求元のサイトによってパーティショニングされていることを確認するためのキャッシュキーとして、firefox [^6]のようなタプル（`(top-frame-site, resource-url)` または chromium/chrome [^5] のような `(top-frame-site, framing-site, resource-url)` ）を使って行われます。
これにより、攻撃者が異なるサイトのキャッシュされたコンテンツを操作することがより困難になります [^1] [^2] [^3]。
Safariは現在、パーティション化されたキャッシュを搭載しています[^4]。

{{< hint tip >}}
パーティション化されたキャッシュを使用しないブラウザの場合、アプリケーションがcache probing技術を対策するために展開できる[other defenses]({{< ref "../design-protections/cache-protections.md" >}})があります。
また、cache probing攻撃を対策するために、あるレベルのユーザインタラクションを必要とするように [designed]({{< ref "../design-protections/subresource-protections.md" >}}) することも可能です。
{{< /hint >}}

## その他の関連プロジェクト

### WebKitのトラッキング防止技術

Safari は `(top-frame-site, resource URL)` をキャッシュキーとして使用するパーティション化された HTTP キャッシュを実装しています。
これは、WebKit のより大きな [Tracking Prevention](https://webkit.org/tracking-prevention/) プロジェクトの一部です。

### FirefoxのFirst Party Isolation

First Party Isolationは、Firefoxの [browser extension](https://addons.mozilla.org/en-US/firefox/addon/first-party-isolation/) で、ドメインごとにクッキーや永続的なデータ（キャッシュなど）へのアクセスを制限するものです。
これには、ユーザ側でのオプトインが必要です。

## 考察

パーティション化されたHTTPキャッシュは、いずれすべてのブラウザに搭載されるであろう有望なセキュリティ機能です。
これらのパーティショニング戦略は、ブラウザのキャッシュを活用するXS-Leakの手法のほとんどを軽減します。
将来的には、パーティション化されたキャッシュは他のブラウザリソースに拡張され、[Socket Exhaustion XS-Leak]({{< ref "../../attacks/timing-attacks/connection-pool.md" >}}) などの他のXS-Leak手法の対策にも役立つかもしれません。

## 参考文献

[^1]: Double-keyed HTTP cache, [link](https://github.com/whatwg/fetch/issues/904)
[^2]: Explainer - Partition the HTTP Cache, [link](https://github.com/shivanigithub/http-cache-partitioning)
[^3]: Client-Side Storage Partitioning, [link](https://privacycg.github.io/storage-partitioning/)
[^4]: Optionally partition cache to prevent using cache for tracking (Webkit), [link](https://bugs.webkit.org/show_bug.cgi?id=110269)
[^5]: Split Disk Cache Meta Bug (Blink), [link](https://bugs.chromium.org/p/chromium/issues/detail?id=910708)
[^6]: Top-level site partitioning (Gecko), [link](https://bugzilla.mozilla.org/show_bug.cgi?id=1590107)
