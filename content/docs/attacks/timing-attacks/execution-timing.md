+++
title = "Execution Timing"
description = ""
date = "2020-10-01"
category = "Attack"
abuse = [
    "Event Loop",
    "Service Workers",
    "Site Isolation",
    "CSS Injections",
    "Regex Injections",
    "iframes",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
    "COOP",
    "Framing Protections",
]
menu = "main"
weight = 3
+++

ブラウザ上でのJavaScriptの実行時間を測定することで、攻撃者は特定のイベントがいつ発生したか、ある操作にどれくらいの時間がかかったかといった情報を得ることができます。

## イベントループのタイミング

JavaScriptの並行処理モデルは、[single-threaded event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)に基づいており、これは一度に一つのタスクしか実行できないことを意味します。
例えば、ある時間のかかるタスクがイベントループをブロックした場合、UIスレッドが枯渇した結果として、ユーザーはページがフリーズしたように感じることがあります。
他のタスクはブロックされたタスクが終了するまで待たなければなりません。
各ブラウザは異なる[プロセスモデル](https://www.chromium.org/developers/design-documents/process-models)を実装しているので、ウェブサイトによっては、その関係によって異なるスレッド（およびイベントループ）で実行されることがあります。

このモデルを悪用して、クロスオリジンページから秘密を盗みだす手法があります。

- イベントプールで次に実行されるまでの時間を測定することで、異なるオリジンのコードが実行されるまでの時間を推測する。攻撃者は固定されたプロパティを持つイベントをイベントループ[^1][^2]に送り続け、プールが空になると最終的にディスパッチされます。他のオリジンは同じプールにイベントをディスパッチし、ここで攻撃者はそのタスクの一つで遅延が発生したかどうかを検出することで、時間差を推測します。
- 攻撃者が制御する文字列によって前記の機密情報が比較される場合、クロスオリジンページから機密情報を盗む。このリークは、1文字ずつの文字列比較[^2]のイベントループで時間差を比較した結果です（前出の手法を使用）。[process isolation](https://www.chromium.org/Home/chromium-security/site-isolation)を行わないブラウザでは、異なるオリジン間のクロスウィンドウ通信が同じスレッドで実行され、同じイベントループを共有します。

{{< hint important >}}
process isolationの仕組みがあるブラウザでは、後者の攻撃の可能性はすでにありません。このような仕組みは、現在Chromiumベースのブラウザでは [Site Isolation](https://www.chromium.org/Home/chromium-security/site-isolation)のみで、Firefox には[Project Fission](https://wiki.mozilla.org/Project_Fission) という名前で*間もなく*導入される予定です。
{{< /hint >}}

## Busy Event Loop

別の手法として、スレッドのイベントループをブロックし、イベントループが再び利用可能になるまでの時間を計測する方法があります。
この手法の主な利点の1つは、攻撃者のオリジンが他のオリジンの実行に影響を与えることができるため、Site Isolationを回避できることです。この攻撃は次のように動作します。

1. 対象のウェブサイトを`window.open`で別ウィンドウに表示するか、`iframe`内に表示する。([Framing Protections]({{< ref "../../defenses/opt-in/xfo.md" >}})が設定されていない場合)
2. 長い計算が始まるのを待つ。
3. [Framing Protections]({{< ref "../../defenses/opt-in/xfo.md" >}})に関係なく、同じサイトのページを`iframe`内に読み込む

攻撃者は、(手順3の)`iframe`が`onload`イベントをトリガーするのにかかった時間を計ることで、ターゲットのウェブサイトが実行された時間を検出できます。([Network Timing]({{< ref "network-timing.md" >}}) of step 3 should be minimal)
両方のナビゲーションは同じコンテキストで発生し、それらは同じサイトであるため、同じスレッドで実行され、同じイベントループを共有します。（それらは互いにブロックすることができます。）

```javascript
// 新しいウィンドウを開いて、example.comのイベントループを
// ウィンドウがブロックする時間を測定する
window.open('https://example.org/expensive');

// TODO: タイムアウトなどでコストの掛かるウィンドウがロードされるのを待ちます。
var ifr = document.createElement('iframe');
ifr.src = "https://example.org";
document.body.appendChild(ifr);

// 初期時間の測定
var start = performance.now();

ifr.onload = () => {
    // iframeが読み込まれたら、時間差を計算する
    var time = performance.now() - start;
    console.log('It took %d ms to load the window', time);
}
```

## サービスワーカー

[サービスワーカー](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)は、ウェブアプリケーションにオフラインソリューションを提供するために使用できますが、攻撃者によってJavaScriptの実行タイミングを計測するために悪用される可能性があります[^4]。サービスワーカーは、ブラウザとネットワークの間のプロキシとして機能し、アプリケーションがメインスレッド（document）によるあらゆるネットワークリクエストを傍受することを可能にします。

タイミングを測定するために、攻撃者は以下の手順を実行することができます。

1. 攻撃者は自分のドメイン(attacker.com)の1つにサービスワーカーを登録する。
2. メインドキュメントで、攻撃者はターゲットウェブサイトへのナビゲーション（window.open）を発行し、サービスワーカーにタイマーを起動するよう指示する。
3. 新しいウィンドウのロードが始まると、手順2で取得した参照をサービスワーカーが処理するページに移動させます。
4. 手順3で実行したリクエストがサービスワーカーに到着すると、サービスワーカーは204（No Content）レスポンスを返し、ナビゲーションを中止する。
5. この時点で、サービスワーカーは、ステップ2で開始したタイマーの計測値を収集します。この測定値は、JavaScriptがどれくらいの時間ナビゲーションをブロックしたかに影響されます。

実際にはナビゲーションは発生しないので、ステップ3から5を繰り返して、連続したJavaScriptの実行タイミングについてより多くの測定値を得ることができます。


### jQuery、CSSセレクタ、そしてShort-circuit Timing

攻撃者は、CSS セレクタのもう一つの興味深い挙動である、式の`short-circuit`（短絡評価）を悪用することができます。この式は`URL`ハッシュで受け取られ、ページ`jQuery(location.hash)`[^3]を実行したときに評価されます。

セレクタ `main[id='site-main']` がマッチせず評価に失敗すると、実行に時間のかかるセレクタの他の部分 (`*:has(*:has(*:has(*))))`) は無視されます (`and` 演算子と同じような感じですが、逆です) ので、タイミング攻撃が可能です。

```javascript
$("*:has(*:has(*:has(*)) *:has(*:has(*:has(*))) *:has(*:has(*:has(*)))) main[id='site-main']")
```

{{< hint tip >}}
process isolation機構を持つブラウザでは、[サービスワーカー]({{<ref "execution-timing.md#service-workers">}})を悪用して実行タイミング計測を取得したり、[Busy Event Loop tricks]({{< ref "#busy-event-loop" >}})などでprocess isolationを回避できます。
{{< /hint >}}


## ReDoS

{{< hint warning >}}
この一連のXS-Leaksは、ターゲットページにRegexを注入することを必要とします。
{{< /hint >}}

正規表現サービス拒否（ReDoS）とは、ユーザー入力として正規表現を許可しているアプリケーションにおいて、サービス拒否を引き起こす手法です [^2] [^5]。悪意を持って細工された正規表現を指数関数的な時間で実行させることができます。これは、ページ上のデータによって異なる実行時間を持つ正規表現を注入することができる場合、XS-リークのベクトルとして使用することができます。これは、クライアントサイドまたはサーバーサイドで起こり得ます。


## 対策

| Attack Alternative | [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |    [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})    |
| :----------------: | :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :-------------------------------------------------------------------------: |
|   T. Event Loop    |                                         ❌                                          |                          ❓                          |                                 ❌                                 | [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |
|  Service Workers   |                                         ✔️                                          |                          ✔️                          |                                 ❌                                 | [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |
|       jQuery       |                                         ✔️                                          |                          ❌                          |                                 ❌                                 | [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |
|       ReDoS        |                                         ✔️                                          |                          ❌                          |                                 ❌                                 | [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |
|  Busy Event Loop   |                                         ✔️                                          |                          ✔️                          |                                 ❌                                 | [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |


## 参考文献

[^1]: Loophole: Timing Attacks on Shared Event Loops in Chrome, [link](https://www.usenix.org/system/files/conference/usenixsecurity17/sec17-vila.pdf)
[^2]: Matryoshka - Web Application Timing Attacks (or.. Timing Attacks against JavaScript Applications in Browsers), [link](https://sirdarckcat.blogspot.com/2014/05/matryoshka-web-application-timing.html)
[^3]: A timing attack with CSS selectors and Javascript, [link](https://blog.sheddow.xyz/css-timing-attack/)
[^4]: Security: XS-Search + XSS Auditor = Not Cool, [link](https://bugs.chromium.org/p/chromium/issues/detail?id=922829)
[^5]: A Rough Idea of Blind Regular Expression Injection Attack, [link](https://diary.shift-js.info/blind-regular-expression-injection/)
