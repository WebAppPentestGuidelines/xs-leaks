+++
title = "Clocks"
description = ""
date = "2020-10-01"
category = "Instrument"
menu = "main"
weight = 1
+++

クロックには、明示的なものと暗黙的なものの2種類があります。
明示的なクロックは、開発者が直接タイミングを測定するために使用されるもので、その機構はブラウザによって明示的に提供されます。
一方で、暗黙的なクロックは、特定のWebの機能を利用して作り出される想定外のもので、それを利用することで相対的な時間経過を測定できるものです。

## 明示的なクロック

### performance.now API

[performance.now()](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now) APIは、開発者がより高精度に時間を計測することを可能にします。

{{< hint info >}}
XS-Leaksのいくつかのタイプを緩和するため、`performance.now()`の精度は、すべてのモダンブラウザでナノ秒からマイクロ秒の範囲に縮小されました。[^1][^2][^3]
<!-- TODO: "to mitigate some" means Size XS-Leaks that were fixed -->

[^1]: Reduce resolution of performance.now (Webkit). [link](https://bugs.webkit.org/show_bug.cgi?id=146531)
[^2]: Reduce precision of performance.now() to 20us (Gecko). [link](https://bugzilla.mozilla.org/show_bug.cgi?id=1427870)
[^3]: Reduce resolution of performance.now to prevent timing attacks (Blink). [link](https://bugs.chromium.org/p/chromium/issues/detail?id=506723)
{{< /hint >}}

### Date API

[Date] APIは、タイミング測定に使用できる最も古いブラウザのAPIです。
これにより開発者は日付を取得したり、`Date.now()`を使ってUnixタイムスタンプを取得したりすることができます。
より新しいAPIが導入される前は、このAPIが攻撃に使われていました。[^3]

## 暗黙的なクロック

### SharedArrayBufferとWeb Workers

`Web Workers`の導入に伴い、スレッド間でデータを交換するための新しいメカニズムが作られました。[^1]それらの機構の一つが`SharedArrayBuffer`で、メインスレッドとワーカスレッドの間でメモリ共有を提供します。悪意のあるウェブサイトは、バッファ内の数値をインクリメントさせる無限ループを実効するワーカーをロードすることで、implicit クロックを作成することができます。この値は、メインスレッドからいつでもアクセスでき、何回インクリメントが行われたかを読み取ることができる。

{{< hint info >}}
[Spectre](https://spectreattack.com)の公開に伴い、`SharedArrayBuffer`はブラウザから削除されました。
その後、2020年に再導入され、このAPIを利用する際はドキュメントが[セキュアコンテキスト](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer)内にあることが要求されるようになりました。
セキュアコンテキストは、アクセスすることを明示的にオプトインしていないクロスオリジンコンテンツは参照できないので、これはSharedArrayBufferがいくつかのXS-Leakにおいてはクロックとして使用できないことを意味します。

モダンブラウザで`SharedArrayBuffer`を使用するには、アプリケーションは以下のヘッダを設定することで、明示的に[COOP]({{< ref "../../defenses/opt-in/coop.md" >}})や[COEP](https://web.dev/coop-coep/)を有効にする必要があります。

```http
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```
{{< /hint >}}


```javascript
// WebWorker内部で実行する関数を定義する
function worker_function() {
  self.onmessage = function (event) {
    const sharedBuffer = event.data;
    const sharedArray = new Uint32Array(sharedBuffer);

	// uint32の数値を無限に増加増加させる
    while (true) Atomics.add(sharedArray, 0, 1);
  };
}

// Create the WebWorker from the JS function and invoke it
const worker = new Worker(
  URL.createObjectURL(
    new Blob([`(${worker_function})()`], {
      type: "text/javascript"
    }))
);

// WebWorkerとドキュメント間のShared bufferを作成する
const sharedBuffer = new SharedArrayBuffer(Uint32Array.BYTES_PER_ELEMENT);
const sharedArray = new Uint32Array(sharedBuffer);
worker.postMessage(sharedBuffer);
```

{{< hint tip >}}
メインスレッドでの相対時間を取得するには、[Atomics API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Atomics)を利用する。
```javascript
Atomics.load(sharedArray, 0);
```

{{< /hint >}}


### その他のクロック

攻撃者が暗黙的なクロックを作り出すために悪用できるAPIは、相当数存在します。：
[Broadcast Channel API](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API)、[Message Channel API](https://developer.mozilla.org/en-US/docs/Web/API/MessageChannel)、[requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)、[setTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout)、CSSアニメーション、その他[^2] [^4].

## 参考文献

[^1]: Shared memory: Side-channel information leaks, [link](https://github.com/tc39/ecmascript_sharedmem/blob/master/issues/TimingAttack.md)
[^2]: Fantastic Timers and Where to Find Them: High-Resolution Microarchitectural Attacks in JavaScript, [link](https://gruss.cc/files/fantastictimers.pdf)
[^3]: Exposing Private Information by Timing Web Applications, [link](http://crypto.stanford.edu/~dabo/papers/webtiming.pdf)
[^4]: Trusted Browsers for Uncertain Times, [link](https://www.usenix.org/system/files/conference/usenixsecurity16/sec16_paper_kohlbrenner.pdf)
