+++
title = "Network Timing"
description = ""
date = "2020-10-01"
category = [
    "Attack",
]
abuse = [
    "iframes",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
    "COOP",
    "Framing Protections",
]
menu = "main"
weight = 2
+++

ネットワーク タイミングのサイドチャネルは、ウェブが誕生したときから存在していました。これらの攻撃は、ブラウザが[performance.now()]({{< ref "clocks.md#performancenow" >}})のような高精度のタイマーを出荷し始めたときに、新しい注目を集め、時とともに異なるレベルの影響を持つようになりました。

タイミング測定値を得るために、攻撃者は暗黙的または明示的なクロックを使用する必要があります。これらの[クロック]({{< ref "clocks.md" >}})は、XS-Leaksの目的では通常交換可能であり、精度と利用可能性が異なるだけです。簡単のために、この記事では、すべてのモダンブラウザに存在する明示的なクロックである`performance.now()`APIを使用することを想定しています。

このサイドチャネルにより、攻撃者はクロスサイトリクエストが完了するまでにかかった時間から情報を推測することができます。ネットワークのタイミング測定は、ユーザーの状態によって変化する可能性があり、通常は下記の要素に連動します。

- リソースサイズ
- バックエンドでの計算時間
- サブリソースの数とサイズ
- [キャッシュの状態]({{< ref "../cache-probing.md" >}}).

{{< hint tip >}}
クロックの種類については、[クロックの記事]({{< ref "clocks.md" >}})で詳しく説明しています。
{{< /hint >}}

## モダンなWebのタイミング攻撃

[performance.now()]({{< ref "clocks.md#performancenow" >}})は、リクエストの実行にかかる時間を測定するために使用できます。

```javascript
// クロックを開始します。
var start = performance.now()

// fetchリクエストが完了するまでの時間を計測します。
fetch('https://example.org', {
  mode: 'no-cors',
  credentials: 'include'
}).then(() => {
  // fetch終了した時点で時間差を計算します。
  var time = performance.now() - start;
  console.log("The request took %d ms.", time);
});
```
## Onloadイベント

同じような処理で、リソースを取得するのにかかる時間を測定するには、単に `onload` イベントを監視することで可能です。

```javascript
// 時間が欲しいページを指すscript要素を作成します。
var script = document.createElement('script');
script.src = "https://example.org";
document.body.appendChild(script);

// クロックを開始します。
var start = performance.now();

// スクリプトがロードされたら、リクエストが完了するまでの時間を計算します。
script.onload = () => {
  var time = performance.now() - start;
  console.log("The request took %d ms.", time)
}
```
{{< hint tip >}}
同様の手法は他の HTML 要素、例えば `<img>`, `<link>`, `<iframe>` にも使うことができ、他の手法が失敗するシナリオで使用することができます。例えば、[Fetch Metadata]({{< ref "/docs/defenses/opt-in/fetch-metadata.md">}})が script タグへのリソースの読み込みをブロックする場合、image タグへの読み込みを許可することがあります。
{{< /hint >}}
{{< hint tip >}}
別の方法として、`image.complete`プロパティを使用することもできます。詳しくは[こちら](https://riccardomerlano.github.io/xs-leaks/cache-probing-through-image.complete-property/)
{{< /hint >}}

## Cross-windowなタイミング攻撃

攻撃者は、`window.open`で新しいウィンドウを開き、`window`の読み込みが始まるのを待つことで、ページのネットワークタイミングを測定することも可能です。以下のスニペットは、この測定の方法を示しています。

```javascript
// 新しいウィンドウを開き、iframeの読み込みを開始するタイミングを測定します。
var win = window.open('https://example.org');
// 最初の時間を計測します。
var start = performance.now();
// ループを定義します。
function measure(){
  try{
    // ページがロードされた場合、そのページは異なるオリジンになるので、
	// `win.origin`は例外をスローします。
    win.origin;
    // ウィンドウがsame-originのままであれば、すぐにループを繰り返しますが、
	// イベントループはブロックしません。
    setTimeout(measure, 0);
  }catch(e){
    // ウィンドウが読み込まれたら、時間差を計算します。
    var time = performance.now() - start;
    console.log('It took %d ms to load the window', time);
  }
}
// ウィンドウのオリジンが切り替わった時点で抜けるループを開始する
measure();
```
{{< hint note >}}
このPOCでは`setTimeout`を使って、 `while(true)`ループに相当する部分を大まかに作成していることに注意してください。JSのイベントループがブロックされるのを避けるために、このような方法で実装する必要があります。
{{< /hint >}}
{{< hint tip >}}
この手法は、[イベントループをビジー状態にする]]({{< ref "execution-timing.md#busy-event-loop" >}})ことで、ページの実行タイミングを測定することにも応用できる。
{{< /hint >}}

## イベントのアンロード

[`unload`](https://developer.mozilla.org/en-US/docs/Web/API/Window/unload_event)と[`beforeunload`](https://developer.mozilla.org/en-US/docs/Web/API/Window/beforeunload_event)イベントは、リソースを取得するのにかかる時間を測定するために使用することができます。これは、ブラウザが新しいナビゲーションを要求したときに`beforeunload`がトリガーされ、一方、そのナビゲーションが実際に発生したときにunloadがトリガーされるためです。この動作により、これら2つのイベント間の時間差を計算し、ブラウザがリソースの取得を完了するまでにかかった時間を測定することが可能です。

{{< hint info >}}
`unload`と`beforeunload`の時間差は`x-frame-options` (XFO)ヘッダーの影響を受けません。なぜなら、このイベントはブラウザがレスポンスヘッダーを認識する前に起動されるからです。
{{< /hint >}}

以下のスニペットでは、[SharedArrayBufferクロック]({{< ref "clocks.md#sharedarraybuffer-and-web-workers" >}})を使用していますが、このスニペットを実行する前に、クロックを開始する必要があります。
```javascript
// WebWorkerが使用するShared bufferの作成
var sharedBuffer = new SharedArrayBuffer(Uint32Array.BYTES_PER_ELEMENT);
var sharedArray = new Uint32Array(sharedBuffer);

// WebWorkerを起動し、呼び出します。
worker.postMessage(sharedBuffer);

var start;
iframe.contentWindow.onbeforeunload = () => {
  // ナビゲーション中の「時間」を取得します
  start = Atomics.load(sharedArray, 0);
}
iframe.contentWindow.onpagehide = () => {
  // ナビゲーション後の「時間」を取得します
  var end = Atomics.load(sharedArray, 0);
  console.log('The difference between events was %d iterations', end - start);
};
```

{{< hint tip >}}
[SharedArrayBufferのクロック]({{< ref "clocks.md#sharedarraybuffer-and-web-workers" >}})は高解像度のタイマーを作成するために使用されました。しかし、iframeの `beforeunload` と `unload` イベント間の時間差は、他のクロック、例えば*performance.now()*でも測定できます。
{{< /hint >}}
{{< hint tip >}}
このスニペットでは、iframeを利用して計測しています。この攻撃のバリエーションは、ウィンドウの参照を使用することもできますが、これに対する防御はより困難です。

{{< /hint >}}
## サンドボックス化されたフレームのタイミング攻撃

もしページに[Framing Protections]({{< ref "../../defenses/opt-in/xfo.md" >}})が実装されていなければ、攻撃者はページとすべてのサブリソースがネットワーク上でロードされるまでの時間を計ることができます。デフォルトでは、iframeの `onload` ハンドラはすべてのリソースがロードされ、すべてのJavaScriptの実行が終了した後に呼び出されます。しかし、攻撃者は `<iframe>` に `sandbox` 属性を含めることで、スクリプト実行時のノイズを除去することができます。この属性はJavaScriptの実行を含む多くの機能をブロックし、その結果、ほとんど純粋なネットワーク計測を行うことができます。

```javascript
var iframe = document.createElement('iframe');
// 対象のWebサイトのURLを設定する
iframe.src = "https://example.org";
// スクリプトの実行をブロックするsandbox属性を設定する
iframe.sandbox = "";
document.body.appendChild(iframe);

// 要求が開始されるまでの時間を測定する
var start = performance.now();

iframe.onload = () => {
  // iframeが読み込まれたら、時間差を計算する
  var time = performance.now() - start;
  console.log("The iframe and subresources took %d ms to load.", time)
}
```

## タイムレスタイミング攻撃

この他に、タイミング攻撃を実行するために時間の概念を考慮しないタイプの攻撃もある。このタイムレス攻撃は、2つのHTTPリクエスト(baseline request及びattacked request)を1つのパケットにまとめ、それらをサーバーに同時に到着させることで成立します。サーバーはリクエストを同時に処理し、その実行時間に基づいたレスポンスを可能な限り最速で返します。2つのリクエストのうちどちらかが先に到着することになり、攻撃者はリクエストの到着順序を比較することで時間差を推測することができます。

この手法の利点は、他の手法では常に存在する、ネットワークのジッターや不確定な遅延から独立していることです。


この攻撃は HTTP の特定のバージョンと共同シナリオに限定されます。それは特定の仮定をし、サーバーの動作に関する要件を持っています。


Other types of attacks do not consider the notion of time to perform a timing attack [^3]. Timeless attacks consist of fitting two `HTTP` requests (the baseline and the attacked request) in a single packet, to guarantee they arrive to the server at the same time. The server *will* process the requests concurrently, and return a response based on their execution time as soon as possible. One of the two requests will arrive first, allowing the attacker to infer the time difference by comparing the order in which the requests arrived.

The advantage of this technique is the independence from network jitter and uncertain delays, something that is always present in the remaining techniques.

{{< hint important >}}
この攻撃は、HTTPの特定のバージョンと共同シナリオに限定されています。特定の前提や、サーバの動作に関して満たさなくてはならないことがあります。
{{< /hint >}}

## 対策

|   Attack Alternative   | [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |                                          [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})                                          |
| :--------------------: | :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------: |
| Modern Timing Attacks  |                                         ✔️                                          |                          ❌                          |                                 ❌                                 | [RIP]({{< ref "/docs/defenses/isolation-policies/resource-isolation" >}}) 🔗 [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |
| Frame Timing (Network) |                                         ✔️                                          |                          ❌                          |                                 ❌                                 |                                        [FIP]({{< ref "/docs/defenses/isolation-policies/framing-isolation" >}})                                         |
| Frame Timing (Sandbox) |                                         ✔️                                          |                          ❌                          |                                 ❌                                 |                                        [FIP]({{< ref "/docs/defenses/isolation-policies/framing-isolation" >}})                                         |
|  Cross-window Timing   |                                         ❌                                          |                          ✔️                          |                                 ❌                                 |                                       [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}})                                       |
|    Timeless Timing     |                                         ✔️                                          |                          ✔️                          |                                 ❌                                 |                                                                            ❓                                                                            |

🔗 – Defense mechanisms must be combined to be effective against different scenarios.


## 参考文献

[^1]: Exposing Private Information by Timing Web Applications, [link](https://crypto.stanford.edu/~dabo/papers/webtiming.pdf)
[^2]: The Clock is Still Ticking: Timing Attacks in the Modern Web - Section 4.3.3, [link](https://tom.vg/papers/timing-attacks_ccs2015.pdf)
[^3]: Timeless Timing Attacks: Exploiting Concurrency to Leak Secrets over Remote Connections, [link](https://www.usenix.org/system/files/sec20-van_goethem.pdf)
[^4]: Cross-domain search timing, [link](https://scarybeastsecurity.blogspot.com/2009/12/cross-domain-search-timing.html)
