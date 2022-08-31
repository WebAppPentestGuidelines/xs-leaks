+++
title = "Hybrid Timing"
description = ""
date = "2020-10-01"
category = "Attack"
abuse = [
    "iframes",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
    "COOP",
]
menu = "main"
weight = 3
+++

ハイブリッド・タイミング攻撃では、攻撃者は最終的なタイミング測定に影響を与える一連の要素の合計を測定することができます。これらの要素には以下のものがあります。

- [ネットワークの遅延]({{< ref "network-timing.md" >}})
- ドキュメントのパース
- サブリソースの検索と処理
- [コード実行]({{< ref "execution-timing.md" >}})

アプリケーションによって値が異なる要素もあります。つまり、[Network Timing]({{< ref "network-timing.md" >}}) はバックエンドの処理が多いページでより重要かもしれませんし、一方 [Execution Timing]({{< ref "execution-timing.md" >}}) はブラウザ内でデータを処理し表示するアプリケーションでより重要かもしれないのです。攻撃者は、より正確な測定値を得るために、これらの要因のいくつかを排除することも可能です。例えば、攻撃者はページを `iframe` として埋め込むことによってすべてのサブリソースを事前にロードし（ブラウザにサブリソースをキャッシュさせる）、その後、それらのサブリソースの取得によって生じる遅延を除外した2度目の計測を行えます。


##  Frameタイミング攻撃(Hybrid)

ページが[Framing Protections]({{< ref "../../defenses/opt-in/xfo.md" >}})を設定していない場合、攻撃者はすべての要素を考慮したハイブリッドな計測結果を取得できます。この攻撃は[Network-based Attack]({{< ref "network-timing.md#frame-timing-attacks-network" >}})と似ていますが、リソースが取得されると、ブラウザによってページがレンダリングされて実行されます（サブリソースの取得と JavaScript の実行が行われる）。このシナリオでは、`onload`イベントは（サブリソースとスクリプトの実行を含めて）ページが完全にロードされたときだけトリガーされます。

```javascript
var iframe = document.createElement('iframe');
// 配送先のWebサイトのURLを設定する
iframe.src = "https://example.org";
document.body.appendChild(iframe);

// リクエストが初期化されるまでの時間を計測する
var start = performance.now();

iframe.onload = () => {
  // iframeが読み込まれたら、時間差を計算する
  var time = performance.now() - start;
  console.log("The iframe and subresources took %d ms to load.", time)
}
```

## 対策

|  Attack Alternative   | [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |  [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})   |
| :-------------------: | :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :----------------------------------------------------------------------: |
| Frame Timing (Hybrid) |                                         ✔️                                          |                          ❌                          |                                 ✔️                                 | [FIP]({{< ref "/docs/defenses/isolation-policies/framing-isolation" >}}) |
